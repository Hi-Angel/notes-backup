# Misc

* usually you get dropped to initram terminal due to lack of drivers to load the system from the partition it's located on. E.g. in case of custom Live DVD/USB it may be lack of drivers for DVD-ROM, even though the initramfs got loaded from it.
* `busybox` is used in initramfs, but it may have some utilities compiled out. That may be dealt with by re-compiling busybox. But since it is for debugging purposes, don't create a whole package out of it *(it may need some installation scripts and whatnot)*, and certainly don't call `make install` which does nothing useful in case of busybox. Instead better manually replace the `busybox` binary that belong to `busybox-initramfs` or similarly named package.
