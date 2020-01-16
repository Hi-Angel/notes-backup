# Terms

* `vdev` *(virtual device)*: a virtual device that may comprise multiple real ones *(e.g. in case of RAID)*.
* `pool`: essentially *(??)* a collection of vdevs.
* `dataset`: basically a directory. Like, a place to store files and other directories, just that.
* `ZVOL` raw block devices, represented to system.
* `L2ARC` and `SLOG`: special kind of created block devices, intended to be fast. Reads and writes to this device are essentially spread over hw disks in a way to maximize speed.
* `scrub`: an utility similar to fsck. It runs against a mounted and live filesystem. Also according to Wiki, ZFS seems to store multiple copies of a file *(disregarding RAID functional)*, so a file whose checksum doesn't match can be restored.
* `quota`: a limit set for a particular dataset. E.g. here we limit a dataset `customers/1` to only have 5G available: `zfs set quota=5G mypool/customers/1`

# Misc

* create a dataset `mydataset`: given existing pool `mypool`, `zfs create mypool/mydataset`
* list datasets: `zfs list`
* Getting all properties for pool `mypool` and dataset `mydataset2`: `zfs get all mypool/mydataset2`.
* Getting id of a dataset: `/lib/udev/zvol_id /dev/zd0`. Example output: `mypool/mydataset2`.

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
