# TCP Offload

Network adapters often support offload to process the network more quickly. The downside though is that it may interfere with packet filtering, firewall rules, etc.

To see the support use `ethtool -k interface | grep offload`. And switching the state is possible with `-K`.

# Misc

* establish mapping between devices in `ip addr` and `lspci`: `lshw -c net -businfo`.
* seeing incoming connections queue on a listening socket *(so called `backlog`)*: `ss -alp` may show them as `Recv-Q` column. It may be one more than the `Send-Q` field, which shows the maximum number of clients that can be accepted. All of that determined through experimentation with a simple tcp server and a bunch of netcat instances connecting to it. After it gets full *(e.g. equal to backlog parameter of `listen()`)*, new connections will get `connection refused`.

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
