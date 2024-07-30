Fiber Channel aka FC. In software it's handled by two competing modules: LIO *(for Linux in-tree)* and SCST *(for Linux out-of-tree)*.

From the outside an FC device is identified by WWPN. On the inside there's no block device *(for SCST at least)*, instead there's a sysfs directory that describes SCST device properties, and then client data go to whatever non-SCST block device is configured as the underlying storage.

# Setting up on a NAS

The following is an example of setting up an arbitrary unnamed NAS just to get the picture.

### Sharing to Windows VM inside a ESXi host

1. Make sure NAS has FC cards in "target" mode *(not "initiator")*
2. Inside NAS web interface create a client and add the client WWPN.

    Gotta chose client WWPNs from a list of "accessible on fabric". It will be the WWPNs that are listed e.g. in ESXi in host setting *(chose a host name in the tree on the left)*, and the tab *(on the right)* `Configure → Storage Adapters`, there will be a table/list where a "Fibre Channel" should be mentioned.
3. Create a pool/volume and assign to it a LUN.
4. Give the client access to the LUN.
5. Go to ESXi, chose the compute cluster that contains VMs in the tree on the left. Then chose tab `Configure`, and then sub-tab `Storage Devices`. The LUN should've appeared in the list there. If it didn't a ESXi reboot may be required.
6. Chose a VM to share the disk to, go to its settings, click `Add new device → RDM device`. Chose the LUN as the device.
7. In Windows guest VM open `diskmgmt`, and mark the disk. More or less as follows:
   1. Open `powershell`, execute `diskpart`, then `list disk`. It shows disk indices. Suppose the disk is `2`. Then execute `select disk 2`, `online disk`, `attributes disk clear readonly`, `create partition primary`. In case you have many disks, these commands may be put to a `1.txt` file each on a separate line and executed as `diskpart /s 1.txt`.

   It is possible with the following `test.ps1` powershell script for disks `[1..9]` *(inclusive)* — but note that it won't abort upon error, so test it with one disk beforehand:

   ```powershell
   1..9 | ForEach-Object {
       echo "
           sel disk $_
           online disk
           attributes disk clear readonly
           create partition primary
       " | diskpart
   }
   ```

   2. *(optional)* assign a letter to the disk and format to NTFS with `select vol 3`, `assign`, `format fs=ntfs quick` commands similarly. It has to be done separately from the actions with partitions because the `vol` index *(can be seen with `list vol`)* may differ. As a script *(the warning mentioned in point 1 applies)*:
   ```powershell
   1..9 | ForEach-Object {
       echo "
           sel vol $_
           assign
           format fs=ntfs quick
       " | diskpart
   }
   ```

# SCST sysfs paths

* `/sys/kernel/scst_tgt/devices/<vol_name>` path to the configuration of a single device. As described at the top, there's no "scst block device", instead a device is an abstract concept identified by various settings in the sysfs dir.
* `/sys/kernel/scst_tgt/devices/<vol_name>/filename` the path to the target block-device, i.e. location for SCST to read/write the data. The device is typically unrelated to SCST otherwise.
* `/sys/kernel/scst_tgt/devices/<vol_name>/active` whether the SCST dev is active.

# Misc

* Optimal and non-optimal paths: a "non-optimal" e.g. "alua-mirror" may get triggered when optimal one is not accessible for some reason. It adds a level of network indirection *(e.g. by passing load through interconnect)*. This ALUA mirror is usually using iscsi protocol *(not FB)*.

  A client always has access to both optimal and non-optimal paths, so technically a client may start using a non-optimal one at any point.
