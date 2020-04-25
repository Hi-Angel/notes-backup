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

Offsets are written in format like `1234567 + 16`, where the left part is the block index and the right one is the number of blocks being acted upon. A block is usually `512` bytes. The offset is apparently in decimals since I've never seen an `abcdef` in it.

# Misc

* block size is usually 512 bytes. AFAIR it was hardcoded. Anyway, it can be gotten with `blockdev --report /dev/sdX`

# RAID misc

## duplicating mount points

I noted it with MD RAID, but may be not specific to it. Suppose you have a RAID over sdX and sdY. Issuing `lsblk -f` may give something like

```
sdX
    md0p0 mount-pointX
sdY
    md0p0 mount-pointX
```

Note the mount point is the same! Devices are different, but mount point is the same. This is actually just a confusing output: you only have one `md0p0` device, and hence one mount point. But the device itself is created over two physical devices sdX and sdY.
