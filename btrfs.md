# Checksum errors/warnings

To check data consistency run `sudo btrfs scrub start -B /mount-path`. There doesn't have to be a RAID for scrub to work *(even though, obviously, data is unlikely to be corrected. It still may correct metadata)*. After scrub is finished, `-B` will make it print stats, but otherwise nothing interesting will be printed. The actual errors/warnings will be printed to system log.

Example:

    [80057.488249] BTRFS warning (device sda4): checksum error at logical 1089945600 on dev /dev/sda4, physical 1098334208: metadata leaf (level 0) in tree 5
    [80057.488269] BTRFS error (device sda4): bdev /dev/sda4 errs: wr 3, rd 748, flush 0, corrupt 26, gen 0
    [80057.490319] BTRFS error (device sda4): fixed up error at logical 1089945600 on dev /dev/sda4

It says, there was an error in metadata, and BTRFS fixed it up.

    [81278.729388] BTRFS warning (device sda4): checksum error at logical 415712260096 on dev /dev/sda4, physical 389413974016, root 5, inode 45992530, offset 0, length 4096, links 1 (path: var/lib/bluetooth/44:03:2C:A2:FB:24/78:A7:EB:7A:3D:E2/info)
    [81278.729419] BTRFS error (device sda4): bdev /dev/sda4 errs: wr 3, rd 748, flush 0, corrupt 27, gen 0
    [81278.729433] BTRFS error (device sda4): unable to fixup (regular) error at logical 415712260096 on dev /dev/sda4

Some data looks corrupted, and BTRFS has no way to restore it *(no RAID in my case)*.

`logical 415712260096` is a logical offset. The file the block belongs to can be retrieved with `btrfs inspect-internal logical-resolve 415712260096 /`.

The next to last line shows counters. They can also be retrieved with `btrfs device stats /dev/sdXn`

* `wr`: `write_io_err`, Idk, but presumably counts failures to write a block. Write error counter increments claimed to be accompanied by kernel log events telling you more.
* `rd`: `read_io_errs`, Idk, but presumably counts failures to read a block.
* `corrupt`: `corruption_errs`, counts found checksum mismatches
* `gen`: `generation_errs`, https://unix.stackexchange.com/questions/285813/what-are-btrfs-generation-errs

## Retrieving a file with broken checksum

This is the most annoying part. Apparently, one can't restore a file on a mounted system. The command will be something like `btrfs restore /dev/sda4 /`; use `-D` for dry-run.

# Misc

* Creating a RAID: `mkfs.btrfs -m raid5 -d raid5 /dev/sd{g,h,i,j,k,m,o,p,r,v}1 -f`. Then to start using it you have to run `mount` command on *any* of the devices, like this `mount -t btrfs /dev/sdg1 /tmp/btrfs-mnt`.
    You may well ask: but can BTRFS RAID provide a raw block device that can be later formatted as `ext4` or whatever. The answer is "No", however you can emulate it by creating a file of required size, and creating a loopback device over it.
* Destroy the RAID: apparently, there's no dedicated command to it. After talking on IRC, the way to get rid of above RAID would be `wipefs -a /dev/sd{g,h,i,j,k,m,o,p,r,v}1`. I was told that btrfs rescan running might be needed, but as of kernel 5.6 I didn't need it, i.e. `btrfs fi show` didn't show the devices anymore.
* creating a snapshot: `btrfs subvolume snapshot / /my-backup-01.03.1871`. This isn't a read-only snapshot, a read-only one could be created with `-r` option.

# Performance

* as of kernel 5.6.2, when a RAID is mounted, space_cache v1 by default is used. My tests on this kernel against mount option `space_cache=v2` consistently shows a bit higher IOPS number. For comparisons, first 3 tests for v1: 64238.6, 53494.3, 50435.4. First 3 tests with v2: 65723.9, 56474.5, 55090.2.
