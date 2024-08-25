Logical Volume Management — allows for representing multiple disks as a single one, an online modifications to partitions, etc. Basically, it's just an added easy-of-use and flexibility.

Layers used in order from the most primitive:

1. **physical volumes**, prefix `pv...` — block devices that LVM is making use of. May be a block device as well as a partition, and loopback device, etc. LVM stores a header into it.
2. **volume groups**, prefix `gv...` — storage pools of **physical volumes** that abstract characteristics of underlying devices.
3. **logical volumes**, prefix `lv...` or `lvm...` — slices of a **volume group**, it's like partitions. Creation example: `lvcreate -L 100M -T vol_group/logical_volume` if thin, `lvcreate -L100M -n logical_volume vol_group` otherwise.

Displaying can be done with `pvdisplay`, `vgdisplay`, `lvdisplay` and `pvs`, `vgs`, `lvs`. The shorter ones usually print info in a more convenient way. The path `/dev/foo/bar` that it displays may not exist if LV status is "not available". For it to appear the logical volume needs to be activated with `lvchange -ay /dev/foo/bar`

## Snapshots

Can't do snapshots of snapshots. Cloning a snapshot is also not currently possible. On IRC people recommend using a `dd` or mounting the partition and copying the content with `rsync`.

Snapshotting is confusing: in LVM you have usual pools, you have thin pool with usual volumes, and then also thin pools with thin volumes… And the snapshotting commands are slightly different for all of them. From what I gather, a "usual snapshot" *(a `lvcreate --size 10G -s --name my_disksnap vol_group/my_disk`)* has to be used for the latter, and a "thin snapshot" *(command `lvcreate -s vol_group/my_disk --type thin`)* for the former.

## Examples:

* Creation:
    ```
    pvcreate /dev/sd{g,h,i,j,k,m,o,p,r,v}1           # mark disks
    vgcreate vol_group /dev/sd{g,h,i,j,k,m,o,p,r,v}1 # create a volume group
    lvcreate -L500G -n disk1 vol_group               # create a disk. It will be at /dev/vol_group/disk1
    ```
* adding a marked disk to a volume group: `vgextend my_vol_group /dev/sda1`
* importing a vol_group: either a `vgchange -ay vol_group` or `vgimport vol_group`

## RAIDs

* Creation: `lvcreate --type raidα -L1G -n my_lv my_vol_group` where raidα is a RAID type.
* RAID status: `lvs -o name,lv_health_status`. Info for various statuses and what to do on those cases may be found in `man lvmraid`.
* Show disks currently used by RAID *(in particular)*: `pvdisplay -m`.

## Misc

* exporting/importing: `vgexport vg` and `vgimport vg`.
  * `activate`ing has to be done after importing a VG: `vgchange -ay vg`, and deactivating before exporting: `vgchange -an vg`.
* adding disks: done separately for group and volume — e.g. `vgextend vol_group /path/to/dev` and `lvextend -l +100%FREE /dev/vol_group/vol` accordingly.
* `pvdisplay -m`: check what devices VGs consist of
* LVM can be made to ignore a device in operations by adding a filter to `/etc/lvm/lvm.conf`. Syntax of the filter is usually mentioned there, just search for `filter` word in comments in the file.
* LVM propagates TRIM/Discard calls on an LV to the underlying physical devices. Worth noting, it is unrelated to `issue_discards` lvm.conf option, which is rather about LVM TRIMing devices after an LV was removed/reduced. In fact, devs doesn't recommend to enable `issue_discards` since it makes one unable to restore data from a removed LV. But at least Debian and Ubuntu enable it anyway.
* if a device with a VG disappeared and re-appeared *(e.g. an usb-stick was taken off/on)*, the VG health status gonna be `error`. I don't know if there's less intrusive way to fix the status, but one that works for me is deactivating, then activating the VG.
* printing specific properties and filtering: use `pvs` and `lvs` with `-o` and `-S` args. Example of printing devices that belong to `my_vg` volume group: `pvs --rows -S "vg_name=my_vg" -o name`.
* `Device /dev/foo not found.` upon running `pvcreate` may mean that the device path is filtered out by `/etc/lvm/lvm.conf`, section `devices { … scan = [permitted_device_paths] …}`
* Display raid-types: `lvs -ao +layout`. The `layout` field should enlist the raid type, but how exactly it works is quite complicated, as it may belong to one of some internal volumes, hence the need in `-a` parameter.
