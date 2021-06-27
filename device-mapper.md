# dmsetup

Allows creating/removing various block devices, stuff like concatenating devices with each another, or creating a faulty device, etc.

## Misc

* create a device mapping, and make it faulty later: first `dmsetup create sdx1 --table '0 300000 linear /dev/sda1 0'` where `0 300000` are sectors, and `sdx1` is the new device name. It will be located at `/dev/mapper/sdx1`. After you're done working with it in a correct state, call `dmsetup -v wipe_table sdx1`
