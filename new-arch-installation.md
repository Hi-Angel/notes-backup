* start downloading arch iso
* burn the iso to a stick
* Format ext4 512Mb for `/boot`, and btrfs `/` for the rest.
* Mount the partitions; enable zstd compression on btrfs.
* Copy `/boot`  to one partition, everything else to another.
* Edit `/etc/fstab` for UUIDs and compression.
* install bootloader
* copy i3 config
* reboot & upgrade
