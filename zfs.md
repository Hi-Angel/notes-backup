# Terms

* `vdev` *(virtual device)*: a virtual device that may comprise multiple real ones *(e.g. in case of RAID)*.
* `pool`: essentially *(??)* a collection of vdevs.
* `dataset`: basically a directory. Like, a place to store files and other directories, just that.
* `ZVOL` raw block devices, represented to system.
* `L2ARC` and `SLOG`: special kind of created block devices, intended to be fast. Reads and writes to this device are essentially spread over hw disks in a way to maximize speed.
* `scrub`: an utility similar to fsck. It runs against a mounted and live filesystem. Also according to Wiki, ZFS seems to store multiple copies of a file *(disregarding RAID functional)*, so a file whose checksum doesn't match can be restored.
