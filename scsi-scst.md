# Misc

* LUN *(Logical Unit Numbers)*: a device, which can be read/written from/to.
* **SCSI initiator**: an endpoint *(e.g. a computer)* that initiates a session against the target
* **SCSI target**: an endpoint that accepts a session from initiators, and provides them with LUNs.
* configuring is done by writing a config file and executing `scstadmin -config myconfig`.
  Internally `scstadmin` configures by writing stuff into various files under `/sys/kernel/scst_tgt`. Stuff similar to the following: `echo "add_target my_target" > /sys/kernel/scst_tgt/targets/iscsi/mgmt`.

## Debugging

Upon rebuilding you do `rmmod iscsi_scst scst` and then `modprobe iscsi_scst`.
