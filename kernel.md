# Installing one module

Use something like

    ```
    make -C /lib/modules/5.4.28-050428-generic/build M=/home/user/Projects/mymodule_dir INSTALL_MOD_DIR=extra/dev_handlers CONFIG_MODULE_SIG_ALL= modules_install
    ```

# Building one module

IIRC the example below worked for me. But there's a gotcha: make sure the module you're building is enabled in configs. For that you can find how the driver is called in configs in `Kconfig` file in its *(or higher)* directory, and then using search in `menuconfig`.

```
make -C . M=drivers/scsi/cxgbi
```

# Errors

* `FATAL: parse error in symbol dump file`: in my experience of building an external module, this means you've got a `Module.symvers` file cached from a different kernel version. If you're building an external module, just run `find -name Module.symvers -delete` over your source code, it should get regenerated.

# Potential improvements, per 4.19 kernel

* `__tcp_transmit_skb` at the end has following code:

    ```
        /* Cleanup our debris for IP stacks */
        memset(skb->cb, 0, max(sizeof(struct inet_skb_parm),
                       sizeof(struct inet6_skb_parm)));
    ```

    I am not sure this is really needed, from the cursory look I am not seeing why would this field have debris; and if function don't need that info anymore, why wouldn't it just ignore it?
* in `tcp_v4_rcv` there's branching upon `sk->sk_state`. The values are enum, but the `sk_state` is a char. See if we can do anything about it.
* `inet_ehashfn` takes some time, the stack ends in there but I'm guessing it's the hashing part that takes time. Can we optimize it somehow?
* in various functions, a bunch of time may be spent to release `skb`: (`skb_release_data`, `kfree`; and the opposite `sk_stream_alloc_skb`). Makes me wonder if we could reserve some hundreds of bytes to make it go easier. UPD: actually, I gotta measure how much time it really takes. It is especially interesting for newer kernel as there was a lot of reworks around memory management.
* (SOLVED) `iscsi_tcp_recv_skb()` has argument `offloaded`, which allows to skip most of the function. It's not quite clear though, what exact feature allows that from the output `ehttool -k`. As of 4.19, the argument is only set in `libcxgbi.c`. Figure out if we can exploit that somehow.
  Answer: no, we can't. Mellanox says it's something like scsi-offload, and indeed it is rarely if ever present on network cards except Chelsio's ones. They also say that performance of such hw depends, but really it's something one would have to bear in mind for any kind of offload for any network card.

# Devices location

* there're devices that appear like `PNP*`, these can be found at `/sys/bus/platform/devices/`
* do `cat /proc/bus/input/devices` to see locations for input devices.

# Block layer

Here's a good article, although it is probably a bit obsolete in the sense that AFAIK page cache for block devices is no more *(there's still VFS one)*. I was told it was removed because all FSes do caching on their own, and that peoples who use raw block devices directly are never interested in having their data cached.

# Debugging

## Measuring time spent

    ktime_t start, end;
    u64 actual_time;
    start = ktime_get();
    // do some workload
    end = ktime_get();
    actual_time = ktime_to_ns(ktime_sub(end, start));
    printk("time spent is %lld ns\n", actual_time);

or, to print accum every second:

```
    static ktime_t prev_sec_at = 0;
    static u64 sum_at_prev_sec_ns = 0;
    ktime_t start, end;
    start = ktime_get();
    // do some workload
    end = ktime_get();
    sum_at_prev_sec_ns += ktime_to_ns(ktime_sub(end, start));
    if (ktime_to_ms(end) - prev_sec_at >= 1000) {
        if (sum_at_prev_sec_ns > 1000000)
            printk("time spent is %lld ms\n", sum_at_prev_sec_ns / 1000000);
        else if (sum_at_prev_sec_ns > 1000)
            printk("time spent is %lld μs\n", sum_at_prev_sec_ns / 1000);
        else
            printk("time spent is %lld ns\n", sum_at_prev_sec_ns);
        prev_sec_at = ktime_to_ms(end);
        sum_at_prev_sec_ns = 0;
    }
```

Note that some of them may be GPL-only if you're building an external module. As far as I can tell, in the kernel type of export is a compile-time decision. But you can edit `META` file of the module you're working with to change license to GPL.

## systemtap (debug live-patching)

Quote: `Systemtap works by converting systemtap scripts into C and building a kernel module that hot-patches the kernel and overrides specific instructions.`. Example: `probe kernel.statement("tcp_ack@./net/ipv4/tcp_input.c:3751") {…code…}`. [Taken from here](https://engineering.skroutz.gr/blog/uncovering-a-24-year-old-bug-in-the-linux-kernel/). Requires no reboot, so it is much faster than throwing `printk`s in a code, rebuilding, reinstalling, rebooting…

# Misc

* while profiling/debugging you may find one of these `ret_from_intr`, `ret_from_exception`, `ret_from_sys_call`, `and` `ret_from_fork`. [There's some description on how they work](https://www.oreilly.com/library/view/understanding-the-linux/0596002130/ch04s08.html), but not much on what they are. I think here's what happens: while a process gets executed, various events may happen. Most usually that is a timer interrupt. At this point kernel scheduler accepts execution. These functions are basically entry points into the scheduler.
