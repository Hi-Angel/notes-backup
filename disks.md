# Automate partitioning a disk

* erase partition table `sgdisk -Z disk`
* print partition table `sgdisk -p disk`
* create partition 1 of size 50k: `sgdisk --new 1::+50K disk`

# Remove a disk

`echo 1 > /sys/block/sdX/device/delete`

# Working with blktrace

* online monitoring `blktrace -d /dev/sda -o - | blkparse -i -`

Otherwise, `blktrace -d /dev/sda` gonna write to `sda.blktrace.*` files, which can later be read with `blkparse sda` or by using an exact file `blkparse sda.blkatrace.0`.

## Reading blkparse output

It's written in man, paragraph `DEFAULT OUTPUT`, but the man is a bit confusing: part of the output, where process name and offsets, is mentioned as some "default header" *(even though it's in the end of each line, rarely a place for headers)*.

Offsets are written in format like `1234567 + 16`, where the left part is offset and the right is the number of blocks being read. It's usually `512` bytes.
