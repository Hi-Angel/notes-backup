# Configuration

Following [this article](https://blog.it-kb.ru/2018/03/12/configure-the-server-with-qlogic-fc-hba-on-debian-linux-9-and-scst-fc-target-as-storage-for-csv-volumes-in-the-hyper-v-cluster-for-highly-available-virtual-machines/), you basically:

1. Make sure `for i in scst{,_vdisk,_disk,_user} qla2xxx_scst qla2x00tgt; do modprobe $i; done` won't give any errors *(whether in terminal or in dmesg)*. For good measure, replace `modprobe` with `echo`, and then copy the output into `/etc/modules`, so it would be loaded on boot.
2. Use `scstadmin -list_target`, `scstadmin -list_handler`, `scstadmin -list_target` to see what you've got. None of these lists should be empty.
3. Create a `/etc/scst.conf` file, where you set options for `HANDLER` and `TARGET_DRIVER`. An example can be seen in `man scst.conf`.
   Note: if you configure `TARGET_DRIVER qla2x00t`, then inside it, in sub-paragraph `TARGET 00:11:…`, where the `00:11:…` is a WWN of FC ports, which can be looked up at `/sys/class/fc_host/host*/port_name`. These params can also be seen at `systool -c fc_host -v` of `sysfsutils` package.
4. Execute `scstadmin -conf /etc/scst.conf`

On errors in dmesg, whenever you see stuff like `sqatgt: Missing parameter foo`, the `foo some_value` has to be added in the `scst.conf`. `scstadmin` will write those into `mgmt` file, but you shouldn't do it manually, there's `scstadmin` for writing there with the right syntax.

# Misc

* LUN *(Logical Unit Numbers)*: a device, which can be read/written from/to.
* **SCST handler** is kinda like an SCST plugin to handle particular type of devices. E.g. to read from block device there's one plugin, from a file system another, and for cdrom there's yet another one. `handler`s are configured in `scst.conf` with a HANDLER section *(see `man scst.conf` for example)*.
* **SCSI initiator**: an endpoint *(e.g. a computer)* that initiates a session against the target
* **SCSI target**: an endpoint that accepts a session from initiators, and provides them with LUNs.
* configuring is done by writing a config file and executing `scstadmin -config myconfig`.
  Internally `scstadmin` configures by writing stuff into various files under `/sys/kernel/scst_tgt`. Stuff similar to the following: `echo "add_target my_target" > /sys/kernel/scst_tgt/targets/iscsi/mgmt`.
* **no `initiator` configuration is required**. While setting up target, you also pass a WWN or IQN or whatever identifier of initiator, and then upon enabling target it will connect to initiator and you shall see a new block device in `lsblk`.

## Typical problems

* **missing target files *(ones with WWN)* at `/sys/kernel/scst_tgt/targets/qla2x00t/`**: check that `/sys/module/qla2xxx_scst/parameters/qlini_mode` is `disabled`. If isn't, add a line `options qla2xxx_scst qlini_mode="disabled"` under a `/etc/modprobe.d` dir and reload the `qla2xxx_scst` driver.

## Debugging

Upon rebuilding you do `rmmod iscsi_scst scst` and then `modprobe iscsi_scst`.
