# TCP Offload

Network adapters often support offload to process the network more quickly. The downside though is that it may interfere with packet filtering, firewall rules, etc.

To see the support use `ethtool -k interface | grep offload`. And switching the state is possible with `-K`.

# Misc

* establish mapping between devices in `ip addr` and `lspci`: `lshw -c net -businfo`.
* seeing incoming connections queue on a listening socket *(so called `backlog`)*: `ss -alp` may show them as `Recv-Q` column. It may be one more than the `Send-Q` field, which shows the maximum number of clients that can be accepted. All of that determined through experimentation with a simple tcp server and a bunch of netcat instances connecting to it. After it gets full *(e.g. equal to backlog parameter of `listen()`)*, new connections will get `connection refused`.
