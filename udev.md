# Basics

See `man 7 udev`.

# Misc

* Rules like `KERNEL` or `SUBSYSTEM` needs to be filled according to the output of `udevadm info --attribute-walk /dev/mydev`. That usually not a full path like you can see in e.g. `udevadm monitor` output.
* udev sets various free-form properties on devices to be later used by applications. E.g. it may set `UDISKS_IGNORE` variable which indicates a suggestion for file-browsers to hide the disk. The variables can be seen with `udevadm info /dev/sdX`
* reloading/retriggering the rules after making change: `udevadm control --reload-rules && udevadm trigger`
* overriding system rules is done by creating a file under `/etc/…` with the same name as the system one. It's the official way recommended in `man udev`. But a more reliable way is just undoing whatever systemd did. Unfortunately, systemd has lots of bugs on that front, see https://github.com/systemd/systemd/issues/24607 and the according PR.

# Debugging

* `udevadm monitor --udev --kernel` will show events happening in the system. That might be not enough to write a rule though, see `KERNEL` example in `Misc` paragraph.
* `udevadm control --log-priority=debug` — sets log priority level to `debug` at runtime. To see the actual messages execute `journalctl -f`.
* `udevadm trigger -v -c add --name-match=/dev/sda` — retriggers kernel and udev events for the device.
