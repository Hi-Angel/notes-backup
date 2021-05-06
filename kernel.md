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

It works by building a kernel module. It is an out-of-tree code, so whether it will build for a newer kernel depends. Note: the module is being built on the first run.

## live-patch

Useful in debugging and development. Works by building a module, which replaces specified function(s) in the kernel *(or its modules I suppose)* to your changed version(s). Requires `CONFIG_LIVEPATCH=y` kernel config set, otherwise it's gonna fail to build with unhelpful compilation errors about missing types.

In my experience, it is convenient to use. Kernel has [`livepatch-sample.c`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/samples/livepatch/livepatch-sample.c) and here's also a Makefile taken [from this great article](https://ruffell.nz/programming/writeups/2020/04/20/everything-you-wanted-to-know-about-kernel-livepatch-in-ubuntu.html) *(recommended reading)*.

```
obj-m := livepatch-sample.o
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)
default:
	$(MAKE) -C $(KDIR) M=$(PWD) modules
clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
```

# DKMS

Allows building external modules.

Notes:

* if you manually build a kernel and install it with `make modules_install && make install`, then make sure that everything DKMS-related is disabled upon `make install`. The problem *(into which I didn't dig)* reproducible at least on Fedora is that `make install` tries to build a DKMS module, and then cleans the whole modules directory, so you basically have to rebuild the kernel all over. It also seems to fail at the end. I seen this reproducible with 2 different modules *(`broadcom-wl-dkms` and `facetimehd`)*, so unlikely it's a bug in packages.
  "How do I disable DKMS" — well, look at `/lib/modules/$(uname -r for newly built kernel)/build/`, it should be a symlink to the kernel build directory *(that is after `modules_install`)*. Remove the link and put a dir, onto which `mount -o ro,bind` the same kernel build directory. Then do `make install`. The dkms will fail, but installation won't. Afterwards, return the symlink back and somehow force the DKMS rebuild *(by reinstalling relevant packages, or in whatever other way)*. It should succeed with no problems now.

# Misc

* while profiling/debugging you may find one of these `ret_from_intr`, `ret_from_exception`, `ret_from_sys_call`, `and` `ret_from_fork`. [There's some description on how they work](https://www.oreilly.com/library/view/understanding-the-linux/0596002130/ch04s08.html), but not much on what they are. I think here's what happens: while a process gets executed, various events may happen. Most usually that is a timer interrupt. At this point kernel scheduler accepts execution. These functions are basically entry points into the scheduler.
