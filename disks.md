# Automate partitioning a disk

## `sgdisk`

* erase partition table `sgdisk -Z disk`
* print partition table `sgdisk -p disk`
* create partition 1 of size 50k: `sgdisk --new 1::+50K disk`

## sfdisk

* create a GPT partition table with one partition of size 500M and the 2nd partition taking the rest of available space
    ```
    sfdisk image.img <<EOF
    label: gpt

    EFI-partition: size=+500M
    +
    EOF
    ```

# Remove a disk

`echo 1 > /sys/block/sdX/device/delete`

# Working with blktrace

* online monitoring `blktrace -d /dev/sda -o - | blkparse -i -`

Otherwise, `blktrace -d /dev/sda` gonna write to `sda.blktrace.*` files, which can later be read with `blkparse sda` or by using an exact file `blkparse sda.blkatrace.0`.

## Reading blkparse output

It's written in man, paragraph `DEFAULT OUTPUT`, but the man is a bit confusing: part of the output, where process name and offsets, is mentioned as some "default header" *(even though it's in the end of each line, rarely a place for headers)*.

Offsets are written in format like `1234567 + 16`, where the left part is the block index and the right one is the number of blocks being acted upon. A block is usually `512` bytes. The offset is apparently in decimals since I've never seen an `abcdef` in it.

## Mapping blktrace/blkparse offset to an actual file

As of writing the words, blkparse shows global indices even if you pointed blktrace at a partition. So:

1. Convert block index to an offset by multiplying by 512
2. Substract offset of the beginning of the partition *(I found it out by using `gdisk -l /dev/sdX` to get partition "start sector", and then multiplied it by 512)*

What to do next varies for file systems. [There's an answer](https://superuser.com/questions/490787/reverse-lookup-of-inode-file-from-offset-in-raw-device-on-linux-and-ext3-4) for ext-family using `debugfs`. For btrfs you translate that to logical address with [this script](https://github.com/Hi-Angel/scripts/blob/master/btrfs-physical-to-logical-mapping.py) and then use it with `btrfs inspect-internal`, for example: `sudo btrfs inspect-internal logical-resolve 378331914240 /`.

# Misc

* block size is usually 512 bytes. AFAIR it was hardcoded. Anyway, it can be gotten with `blockdev --report /dev/sdX`
* `gdisk` shows offsets in sectors, and the sector size is usually 512. As a hint that this is the case note whether first partitions starts at `2048` offset. `2048 * 512 = 1MB`.
* the amount of disk IO may be monitored through means of `/proc/diskstats`, by checking its content before and after doing some IO. The content isn't human-readable, but [this script](https://github.com/Hi-Angel/scripts/blob/master/cmp-diskstats.py) may help. Worth noting though, it seems to not work with `brd` module emulating disks in RAM. The read/write count always stays zero for it, probably worth reporting a bug.
* `ALUA disk` is a virtual devices that is used to access another actual device. The important point is that the device is accessible both as a non-virtual device and a virtual one.

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
