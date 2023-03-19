Creating ramdisk, ramfs.

Usage: `modprobe brd rd_nr=1 rd_size=61440`, where rd_nr is the number of disks to create with the size `61440` of kbytes. Afterwards, first disk is available at `/dev/ram0`, second is at `/dev/ram1`, etc.
