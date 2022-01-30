# Misc

* **do not use `grub-install` without chroot**, unless you're installing a bootloader for the current system, that is. It is a buggy terrible command, which will do tons of useless renames on your current system, and it also will try to detect your current partitions. It completely does not care what disk you're installing bootloader for, it will use your currently loaded disk to figure out *(if you're lucky)* modules to use, thus later you might get a `unknown filesystem` errors from grub. The only way to work around it is to chroot into the system you're installing bootloader for *(so ignore it in case you are already running the system you will be booting)*, and then call `grub-install` from there.
* if you decided to venture on using it without chroot, then calls that proven to have worked in some environments:
  * MBR installation call: `grub-install --boot-directory=/mnt/boot /dev/sdX`. The `/dev/sdX` is a device to install bootloader to.
  * EFI installation call: `grub-install --target=x86_64-efi --efi-directory=/mnt/boot/efi --boot-directory=/mnt/boot/`. No need to append the device to install to, everything is done in terms of directories.

# Prompt

Prompt gives autocompletion. Partitions are seen like `(hd1,…)` stuff like that, use <kbd>TAB</kbd> to see a list.

Booting from it manually requires 3 commands:

```
linux (hdX,…)/path/vmlinuz
initrd (hdX,…)/path/initrd.img
boot
```
