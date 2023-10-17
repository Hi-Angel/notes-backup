# TCP Offload

Network adapters often support offload to process the network more quickly. The downside though is that it may interfere with packet filtering, firewall rules, etc.

To see the support use `ethtool -k interface | grep offload`. And switching the state is possible with `-K`.

# UDP multicast

Multicast is basically when many computers share same IP *(and seems to be only possible with UDP but not TCP due to complications with handshakes and stuff)*. E.g. if you send a message to IP `224.0.55.55` and port `1234`, it will go to each computer that used `bind()` to the port 1234 and the IP *(or alternatively bind allows for `INADDR_ANY`)*, and then joined the group with `224.0.55.55`.

A device joining group `224.0.55.55` sends a IGMP request that notifies the switch that any traffic for the IP should be routed here as well. If "LOOPBACK" socket option is set *(default)*, the device will receive traffic it sent too.

Gotcha: IGMP will not be sent through every network interface of the device, but through only one. In absence of multicast-specific routes it will likely be the default interface. `ip` command is multicast aware, so to find the destination interface you can execute `ip route get 224.0.55.55`. An application can override that with `IP_MULTICAST_IF` and `SO_BINDTODEVICE` socket options. As a separate note, `bind()`ing to a local IP doesn't work the expected way with multicast, but the mentioned options do.

`iperf` also has options for testing multicast, e.g. server `iperf -s -u -B 224.0.55.55 -i 1` and client `iperf -c 224.0.55.55 -u -T 32 -t 3 -i 1`.

IPs are usually taken from the multicast ranges `[224.…] … [239.…]`.

# Misc

* establish mapping between devices in `ip addr` and `lspci`: `lshw -c net -businfo`.
* seeing incoming connections queue on a listening socket *(so called `backlog`)*: `ss -alp` may show them as `Recv-Q` column. It may be one more than the `Send-Q` field, which shows the maximum number of clients that can be accepted. All of that determined through experimentation with a simple tcp server and a bunch of netcat instances connecting to it. After it gets full *(e.g. equal to backlog parameter of `listen()`)*, new connections will get `connection refused`.
* find out network connections of a process: `lsof -i -ap $PID`
    * match them up with FDs: there should be `FD` field, just ignore the `u` letter at the end. Alternatively: first `ls -l /proc/$PID/fd`, then compare the XXXX in `socket:[XXXX]` with DEVICE column in the 1st command.
* `performance`: see `qperf.md`

# vlan

VLANs allow to limit access to a specified group of users by dividing workstations into different isolated LAN segments. Creating is as simple as `ip link add link ethX name my_vlan_name type vlan id 1`. Note: upon adding more than one vlan to the interface you gotta change `id`, otherwise you'll get `VLAN device already exists.`

# Aggregate interfaces aka bonding

Makes multiple NICs look as if they're a single one. Usecases are: 1. increasing bandwidth, 2. keeping connection if an interface went down *(like if you're connected via ethernet and wifi, and then wifi goes down)*. An aggregate supports different modes, the on you may want depends on the usecase. Default mode is usually `round-robin`.

Example of creating one:

```
ip link add bond0 type bond                    # create bond NIC
ip link set bond0 type bond mode active-backup # set a mode
ip link set ethX master bond0                  # add a NIC to bond
```
