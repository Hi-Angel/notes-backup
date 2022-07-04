# Terms

* `vdev` *(virtual device)*: a virtual device is a base for zpools. Usually vdev is a real hw block device. But it's called "virtual" because it also can be a file or partition of a disk.
* `pool`: essentially *(??)* a collection of vdevs.
* `dataset`: basically a directory. Like, a place to store files and other directories, just that.
* `ZVOL` raw block devices, represented to system.
* `scrub`: an utility similar to fsck. It runs against a mounted and live filesystem. Also according to Wiki, ZFS seems to store multiple copies of a file *(disregarding RAID functional)*, so a file whose checksum doesn't match can be restored.
* `quota`: a limit set for a particular dataset. E.g. here we limit a dataset `customers/1` to only have 5G available: `zfs set quota=5G mypool/customers/1`
* `metaslab`: a hash tree that stores what and where blocks are written. It's usually being written from disks, and there's an option to make it always be loaded in RAM for more speed.

# Failing drives

This can be simulated with `zpool offline mypool device`.

For missing drive `zpool status` says "OFFLINE" for the disk and "DEGRADED" as a pool state. It can be replaced with `zpool replace -f mypool old_device new_device`. That will trigger a resilver/rebuild, and the `scan:` in `zpool status` will say about resilver happening/happened.

# Debugging

* `zdb` is a zfs debugger.

## Quickly installing the code while hacking

Code below is for module itself, so adapt for you needs if you're hacking into some userspace utilities.

```
cd module/
make modules && cp zfs/zfs.ko /lib/modules/$(uname -r)/extra/zfs/zfs.ko
```

# Removing older zfs

At least for deb, there's no meta-package for all separate ZFS packages, so upon upgrading a version you may get a bunch of leftovers. You can remove zfs before installing a newer version with something like:

```
apt remove $(dpkg -l | perl -lane 'print @F[1] if /^ii/ and /.*0.8.0-999.*/')
```

# Misc

* Creating a pool `zpool create mypool /dev/disk/by-id/some-disk`.
* Creating a dataset `mydataset`: given existing pool `mypool`, `zfs create mypool/mydataset`
* listing datasets: `zfs list`
* Getting all properties for pool `mypool` and dataset `mydataset2`: `zfs get all mypool/mydataset2`.
* Getting id of a dataset: `/lib/udev/zvol_id /dev/zd0`. Example output: `mypool/mydataset2`.
* Getting `/dev/zd*` device *(at least if the dataset represents a block device)*: `ls -l /dev/zvol/mypool/mydataset`
* `dd: error writing '/dev/zd0': No space left on device` — that error can appear if you ran `dd` *(or similar command)* too early. Per my understanding, after pool and volume has been created, there's a short time before `zd0` is configured. You need to wait, say, a second. I've seen this happening on 0.8.3. IMO this is a bug as commands creating a volume shouldn't return if `zd0` wasn't configured yet, but I don't have motivation in reporting it right now.
* pool load: `zpool iostat -v 1`, shows IO load on individual disks of the pool.

# Sizes

Sizes might be confusing, so here's a little cheatsheet.

* `reservation` on a dataset is an allocation of space from the pool that is guaranteed to be available to the dataset.
* `refquota`: limits the amount of space that the dataset can consume. This hard limit does not include space that is consumed by snapshots and clones.
* `refreservation` reserves space for the dataset that does not include space consumed by snapshots and clones, i.e. for data-only and its metadata. If refreservation is set, a snapshot is only allowed if enough free pool space exists outside of this reservation to accommodate the current number of referenced bytes in the dataset

Useful links https://www.mceith.com/blog/?p=153

# Installing

You can create packages, but there's no single command to only create all packages you may need. It either creates too many or too little.

So, assuming you on deb-based distro *(similar for rpm-based)*, do `make deb`, then `rm *dkms*.deb` *(or remove `kmod` ones if you want dkms instead)*, then install stuff.

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

All writes are done in "transaction groups". Consistency is guaranteed at their granularity. Each group has a number that is referred to in other structs.

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

## DVA

Data Virtual Address, location of the data. `zdb` shows it in bytes, but ZFS internally stores it as 512-sized sector index. A block may contain multiple DVAs *(up to `SPA_DVAS_PER_BP`)*.

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

## Objects

* `bpobj` type of object stands for "block pointer object". Used mainly in snapshots, but from my experiments some of them store prev. system state even without snapshots being used.

## ARC (Adaptive Replacement Cache)

Cache for data, metadata, etc. Async data gets cached in ARC, while sync data gets written to ZIL.

Its stats can be seen at `/proc/spl/kstat/zfs/arcstats`, also there's `arc_summary.py` script that makes that data even more interesting.

While ARC is global for the whole ZFS system, one can add another cache L2ARC, which is specific to a pool.

## References

* https://habr.com/ru/post/160943/
* Inspecting ZFS with radare https://habr.com/ru/post/348354/
