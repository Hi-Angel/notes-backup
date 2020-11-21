# netlink

Used for communication between userspace processes and between a process and a kernel. As an example of the later instance see `man 7 rtnetlink`.

## Downsides

Delivery is not guaranteed, in particular may drop packets if queue is full. This is different from e.g. DBus which would block the caller in this case.
