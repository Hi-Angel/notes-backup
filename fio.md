# Examples

* `fio --name=temp-fio --bs=4k --ioengine=libaio --iodepth=8 --rw=randwrite --filename=/dev/sdX --numjobs=4 --group_reporting --time_based --runtime=20m --output=log_file`
* `fio --name=temp-fio --bs=64k --ioengine=libaio --iodepth=1 --rw=randwrite --rwmixwrite=100 --filename=/dev/sdX --numjobs=1 --group_reporting --size=100G --io_size=1G --output=/tmp/log_file`
