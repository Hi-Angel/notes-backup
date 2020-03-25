# Automate partitioning a disk

* erase partition table `sgdisk -Z disk`
* print partition table `sgdisk -p disk`
* create partition 1 of size 50k: `sgdisk --new 1::+50K disk`

# Remove a disk

`echo 1 > /sys/block/sdX/device/delete`
