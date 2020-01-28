# Terms

* `vdev` *(virtual device)*: a virtual device is a base for zpools. Usually vdev is a real hw block device. But it's called "virtual" because it also can be a file or partition of a disk.
* `pool`: essentially *(??)* a collection of vdevs.
* `dataset`: basically a directory. Like, a place to store files and other directories, just that.
* `ZVOL` raw block devices, represented to system.
* `L2ARC` and `SLOG`: special kind of created block devices, intended to be fast. Reads and writes to this device are essentially spread over hw disks in a way to maximize speed.
* `scrub`: an utility similar to fsck. It runs against a mounted and live filesystem. Also according to Wiki, ZFS seems to store multiple copies of a file *(disregarding RAID functional)*, so a file whose checksum doesn't match can be restored.
* `quota`: a limit set for a particular dataset. E.g. here we limit a dataset `customers/1` to only have 5G available: `zfs set quota=5G mypool/customers/1`
* `metaslab`: a hash tree that stores what and where blocks are written. It's usually being written from disks, and there's an option to make it always be loaded in RAM for more speed.

# Debugging

* `zdb` is a zfs debugger.

# Misc

* Creating a pool `zpool create mypool /dev/disk/by-id/some-disk`.
* Creating a dataset `mydataset`: given existing pool `mypool`, `zfs create mypool/mydataset`
* listing datasets: `zfs list`
* Getting all properties for pool `mypool` and dataset `mydataset2`: `zfs get all mypool/mydataset2`.
* Getting id of a dataset: `/lib/udev/zvol_id /dev/zd0`. Example output: `mypool/mydataset2`.
* Getting `/dev/zd*` device *(at least if the dataset represents a block device)*: `ls -l /dev/zvol/mypool/mydataset`

# What devices constitute a ZFS pool RAID?

If you execute `zpool status -L mypool`, you'll see something like:

```
pool: mypool
 state: ONLINE
  scan: none requested
config:

        NAME                              STATE     READ WRITE CKSUM
        mypool                            ONLINE       0     0     0
          raidz2-0                        ONLINE       0     0     0
            sda                           ONLINE       0     0     0
            sdb                           ONLINE       0     0     0
errors: No known data errors
```

These `someid1` and `someid2` accord to `/dev/disk/by-id/someid1` and `…2` *(note, the id is not uuid)*,

# ZFS internals

Details can be found at "ZFS ondisk format" document.

All writes are done in "transaction groups". Each group has a number that is referred to in other structs.

* `vdev label` is contained within first `256KB` of each physical vdev. It has various information about the current vdev, other vdevs that are in the pool, etc. A vdev has 4 copies of the label.
* `uberblock array` is contained inside the label *(after various other information)*. It contains an info necessary to access the pool content.
    Only one `uberblock` may be active, and it's never updated. Instead write is done into another `uberblock` in the array, and then transaction group number and timestamp are incremented to make it the active one.

Each vdev is broken to "partitions" called `metaslabs`. Usually there's roughly 200 of them.

Each metaslab has `spacemap`.

## Spacemap

Represented as a tree, and allows to work with free and used blocks inside a metaslab. Can be dumped e.g. with `zdb -mmm mypool`.

Block sizes it allows are in range 512..16М. "Free space" doesn't get accounted for blocks that never were used, it only does for those that were written and later removed.

Upon write to some metaslab, ZFS makes up a map of free space "free map", to figure out appropriate ranges of contiguous free space.

## Dataset

`Dataset` in ZFS represents a collection of `object`s. It manages space consumption stats, contains object set location information, and snapshots connections.

An object may represent a bunch of stuff, there are various types of objects. But ultimately, it has a block pointer and number of indirection levels. Object itself is represented by `dnode` struct.

### dataset zdb dump

```
0 L1  0:9800:400 1:9800:400 4000L/400P F=2 B=190
```

 * `L1` means two levels of indirection *(a num of block pointers which need to be traversed to arrive at this data)*. `L0` would mean a `block level`, it's the one that actually hold data.
 * at `0:9800:400`:
   * 0 is some device index
   * 9800 is offset from the beginning of the disk
   * 400 size of the block (0x400 == 1024)
   * also: the `0:9800` is a dva1 *(Data Virtual Address 1)*
 * `1:9800:400` similar to previous: ZFS is using two disk blocks to hold pointers to file data.
 * `F=2` "fill count", it's the number of non-zero block pointers under this block pointer.
   * At `block level`, i.e. at `L0` this counter means if block has data or not *(0 or 1)*.
 * `B=190` birth time, same as txg number (190) that creates the block.

## References

* https://habr.com/ru/post/160943/
