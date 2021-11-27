# Basics

See `man 7 udev`.

# Misc

Rules like `KERNEL` or `SUBSYSTEM` needs to be filled according to the output of `udevadm info --attribute-walk /dev/mydev`. That usually not a full path like you can see in e.g. `udevadm monitor` output.

# Debugging

* `udevadm monitor --udev --kernel` will show events happening in the system. That might be not enough to write a rule though, see `KERNEL` example in `Misc` paragraph.
* `udevadm control --log-priority=debug` — sets log priority level to `debug` at runtime. To see the actual messages execute `journalctl -f`.
* `udevadm trigger -v -c add --name-match=/dev/sda` — retriggers kernel and udev events for the device.
