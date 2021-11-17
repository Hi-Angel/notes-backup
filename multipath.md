# General

Wiki [explains it well](https://en.wikipedia.org/wiki/Multipath_I/O), but basically it's when a device *(usually a storage device)* connected through multiple paths.

May one path fail, there will be no down-time. The system achieves this by providing a single device file despite there be multiple paths. But there are also path-specific device-files *(which you can use e.g. to simulate a device failure)*.

## Example:

```
$ multipath -l /dev/sdag
35000cca02d4a86a4 dm-48 HGST,HUC101812CS4204
size=1.1T features='0' hwhandler='0' wp=rw
`-+- policy='round-robin 0' prio=0 status=active
  |- 11:0:7:0  sdag 66:0   active undef unknown
  `- 11:0:21:0 sdat 66:208 active undef unknown
```

Here `/dev/dm-48` is a device file abstracting two paths into a single one, and `sdag` and `sdat` are path-specific devices.

But what about partitions for `dm-48`?

```
$ fdisk -l /dev/dm-48
…
Device                                  Start       End   Sectors  Size Type
/dev/mapper/35000cca02d4a86a4-part1       256 293020053 293019798  1.1T Linux filesystem
/dev/mapper/35000cca02d4a86a4-part2 293020160 293020175        16   64K Linux filesystem
$ ls -l /dev/mapper/35000cca02d4a86a4-part1
lrwxrwxrwx 1 root root 8 Nov 17 13:42 /dev/mapper/35000cca02d4a86a4-part1 -> ../dm-52
```

So `/dev/dm-52` is a partition of device `/dev/dm-48`.

# Debugging

## Bringing device down and up

Write `offline` to `/sys/block/${DEV}/device/state` to offline the device.

Later to bring it back you need to delete the device *(write `1` to `/sys/block/${DEV}/device/delete`)*, then rescan hosts by writing `- - -` to `/sys/class/scsi_host/${HBA}/scan`, use all HBAs for good measure.

Then for `multipath` to find device newly appeared execute `multipath -v2` *(`-v2` is just verbose output)*
