# TCP Offload

Network adapters often support offload to process the network more quickly. The downside though is that it may interfere with packet filtering, firewall rules, etc.

To see the support use `ethtool -k interface | grep offload`. And switching the state is possible with `-K`.

# Misc

* establish mapping between devices in `ip addr` and `lspci`: `lshw -c net -businfo`.
