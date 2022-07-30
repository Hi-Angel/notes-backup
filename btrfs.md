# Misc

* Creating a RAID: `mkfs.btrfs -m raid5 -d raid5 /dev/sd{g,h,i,j,k,m,o,p,r,v}1 -f`. Then to start using it you have to run `mount` command on *any* of the devices, like this `mount -t btrfs /dev/sdg1 /tmp/btrfs-mnt`.
    You may well ask: but can BTRFS RAID provide a raw block device that can be later formatted as `ext4` or whatever. The answer is "No", however you can emulate it by creating a file of required size, and creating a loopback device over it.
* Destroy the RAID: apparently, there's no dedicated command to it. After talking on IRC, the way to get rid of above RAID would be `wipefs -a /dev/sd{g,h,i,j,k,m,o,p,r,v}1`. I was told that btrfs rescan running might be needed, but as of kernel 5.6 I didn't need it, i.e. `btrfs fi show` didn't show the devices anymore.
* creating a snapshot: `btrfs subvolume snapshot / /my-backup-01.03.1871`. This isn't a read-only snapshot, a read-only one could be created with `-r` option.

# Performance

* as of kernel 5.6.2, when a RAID is mounted, space_cache v1 by default is used. My tests on this kernel against mount option `space_cache=v2` consistently shows a bit higher IOPS number. For comparisons, first 3 tests for v1: 64238.6, 53494.3, 50435.4. First 3 tests with v2: 65723.9, 56474.5, 55090.2.
