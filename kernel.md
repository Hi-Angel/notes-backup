# General

* kernel types in general:
  * monolithic: drivers are in the kernel.
  * microkernel: drivers are userspace. The only components *required* to be in kernel are memory allocation, CPU scheduler and IPC. Ex.: Mach, QNX, Redox.
  * exokernel: allows almost direct access app → hw, only making sure there's no resources conflict *(and perhaps some basic security)*. The app then can either implement the device management on its own or use available library abstractions instead.

## Memory

* kernelspace uses both physical *(aka logical)* and virtual memory. Mostly the latter.
  * `kmalloc` and friends accept GFP flag to describe the purpose, in particular whether the memory can be reclaimed *(aka swapped out)*
* NUMA implies different mem banks attached to different CPU.
* a process information *(memory, opened FDs, in-fly signals)* being kept in `task_struct`
* virtual memory:
  * `Page Table`: a structure to convert virtual → physical addresses. Per process.
  * `PTE`, `Page Table Entry`: a single address mapping in the PT.
  * `TLB`, `Translation Lookaside Buffer`: caches recently used mappings, a part of MMU. It's cleared on process context switch *(unless a CPU supports binding every TLB entry to a PID, e.g. with `PCID` on x86 or `domains` on ARM)*. CPU can only access memory via TLB, so access involves copying a PTE to TLB.
  * "TLB miss": a PTE was not found in TLB, so the CPU goes fetching the mapping to TLB prior to restarting the access. It's different from a "page fault".
  * "page fault": when after a TLB miss the mapping was also not found in PT. It's a valid situation as he physical address may not have been allocated yet or was swapped out.
  * **How**: if TLB lacks a mapping, i.e. a TLB/cache miss, there will be a "page walk", a look up of the mapping in the PT. If it's found, it's written to the TLB because CPU can only access mapping via it, then the faulted instruction restarted.
* `MMIO`, `Memory Mapped IO`: a virtual memory mapped to a device. On the device side it may be contiguous or be represented by registers. Main point: the CPU reads/writes the memory, as opposed to DMA where the device does that.
* `PMIO`, `Port Mapped IO`: no virtual mem is involved, instead special instructions like `in` and `out` are used for access.
* `DMA`: a RAM accessible for reading/writing by a device. The main difference to `MMIO` is CPU does no control besides the initial setup.
* `IOMMU`: due to DMA posing a security problem of hacker-owned device being able to read entire memory, `IOMMU` was created for restricting the device access.

## Other

* most syscalls *(e.g. read)* may be interrupted by sending a signal to the process, e.g. `SIGUSR1`.
  * The order of events: 1. signal comes, 2. syscall interrupted, 3. the process signal handler executed, 4. if process still alive, syscall returns a `EINTR`.
  * in a rare occasion a syscall may be "uninterruptable" *(so nothing including a SIGKILL can make it return)* or "killable" *(like "uninterruptable" but can be killed)*.
* `fork()` is implemented via `clone` syscall. In-kernel it copies `task_struct` and `thread_info`, allocates PID and overwrites some struct fields as necessary. Potentially copies some data per `clones`'s flag.
* threads are implemented via `clone` as well. From kernel POV threads are just processes without the COW stuff.
* `zombie` is a process whose `task_struct` wasn't removed yet.
  * if parent is dead, per `man 2 wait`, the zombie usually gets reassigned to `init`, which periodically uses something similar to `waidpid(-1)` to get rid of zombies. Alternatively, it gets re-assigned to a subreaper process as set by `prctl`.

  Some sources claim that zombie gets reassigned to a grandparent or even to a sibling, but it's not the documented behavior, so probably a mistake.
* process scheduler allocates share of CPU time and enables interactivity by running a process that has slept immediately after it wakes up, thus preempting the long running process that has consumed much larger CPU time share.
* syscalls are done by using either an interrupt *(e.g. int128 on x86)* or a an instruction *(e.g. `systenter` on x86)*.
* `exceptions` *(as opposed to `interrupts`)*: there are 2 types: 1. hw exceptions that happen due to an illegal activity *(e.g. MMU detecting an illegal access)*, 2. OS exceptions, caused by a task, e.g. "divide-by-zero" or similarly accessing invalid memory. Kernel handling for exceptions and interrupts is similar.
* `IRQ` is a number assigned to an interrupt. Can be assigned dynamically or statically.
* kernel processing of an interrupt is divided to top/bottom halves, where top half is the absolutely critical and fast processing *(such as acknowledging the interrupt)* that runs immediately, and bottom half is non-critical time-consuming work *(such as copying network data)* that runs ± "when convenient".
* `/proc/interrupts` enlists info on the available IRQs and their devices.
* Device types: while there're two `block` and `char`, but char ones may have quite complicated API *(such as camera device)* and better be accessed via a special lib. char devs are usually manipulated with `ioctl()` rather than actually written to. Devs are typically located in `/dev` but don't have to. They may be in `/sys` for example.
* `module_init()` is a function initializing various resources and in case the module is compiled-in it's run on boot. OTOH `module_exit()` doesn't run if a module is compiled into the kernel.
* **device model** represents a tree of devices. It serves multiple purposes, one interesting example is powering down devices, which you need to do starting with a leaf, so e.g. 1. usb mouse, 2. usb controller, PCI bus.

    In kernel a device is typically represented by `kobject`, often embedded into another struct. kbojects support reference counting, so each time some code obtains a reference the count is increased. Most often though they're manipulated upon by driver subsystem rather than the driver.

    The kobjects hierarchy is exported to sysfs to facilitate debugging *(among other reasons)*. Though it's not done automatically upon kobject initialization and instead happens explicitly with `kobject_add()`.

     DT enlists drivers compatible with the device via `compatible` keyword.
  * Overlay: an API that allows a driver to modify the DT in runtime. Nodes added should result in the device created, and removing a node should get device de-registered.
* devices may send events *(e.g. "disk full", "low battery", etc)*, which is done via netlink, where the source will be the device sysfs path. Which is facilitated by devices being represented by `kobject`s.
* `DRM`
  * Main GPU devices like `/dev/dri/card1` have a `master`, which is assigned by a `SET_MASTER` ioctl. The master then may "lease" some device resources to other clients.
  * There are "render nodes" like `/dev/dri/renderX` that allow for unprivileged access with no authentication and are useful for render-only purposes *(e.g. off-screen rendering)*.
  * KMS from userspace POV is basically initializing a `struct drm_mode_atomic` and issuing a `DRM_IOCTL_MODE_ATOMIC` ioctl.
* `ACPI` vs `Device Tree`: both provide description of non-enumerable devices and their configuration details. But ACPI additionally abstracts out some knobs, such as power management, power profiles. Also, ACPI is stored in BIOS mem.

# Installing one module

Use something like

```
make -C /lib/modules/5.4.28-050428-generic/build M=/home/user/Projects/mymodule_dir INSTALL_MOD_DIR=extra/dev_handlers CONFIG_MODULE_SIG_ALL= modules_install
```

See also: https://www.kernel.org/doc/Documentation/kbuild/modules.txt

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

## Notes

* if you manually build a kernel and install it with `make modules_install && make install`, then make sure that everything DKMS-related is disabled upon `make install`. The problem *(into which I didn't dig)* reproducible at least on Fedora is that `make install` tries to build a DKMS module, and then cleans the whole modules directory, so you basically have to rebuild the kernel all over. It also seems to fail at the end. I seen this reproducible with 2 different modules *(`broadcom-wl-dkms` and `facetimehd`)*, so unlikely it's a bug in packages.
  "How do I disable DKMS" — well, look at `/lib/modules/$(uname -r for newly built kernel)/build/`, it should be a symlink to the kernel build directory *(that is after `modules_install`)*. Remove the link and put a dir, onto which `mount -o ro,bind` the same kernel build directory. Then do `make install`. The dkms will fail, but installation won't. Afterwards, return the symlink back and somehow force the DKMS rebuild *(by reinstalling relevant packages, or in whatever other way)*. It should succeed with no problems now.

## Misc

* `build` dir in `/lib/modules/$(uname -r)/build` is usually a symlink to a linux-headers package/dir.

# Misc

* while profiling/debugging you may find one of these `ret_from_intr`, `ret_from_exception`, `ret_from_sys_call`, and `ret_from_fork`. [There's some description on how they work](https://www.oreilly.com/library/view/understanding-the-linux/0596002130/ch04s08.html), but not much on what they are. I think here's what happens: while a process gets executed, various events may happen. Most usually that is a timer interrupt. At this point kernel scheduler accepts execution. These functions are basically entry points into the scheduler.
* debian packages can be built from the kernel using `bindeb-pkg` makefile target *(and some others)*
* `kprobe` vs `tracepoint`: `kprobe`s is any place in the kernel, up to a machine instruction within a function *(`bpftace` allows that)*. `tracepoint`s on the other hand are defined statically/explicitly in the code. So basically, new tracepoints can't be added without modifying the source, but kprobes can be any location and does not require kernel to be rebuilt.
