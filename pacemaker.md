Basic design is:

1. One node is active, others are passive
   * The cluster is accessed via virtual IP, so when active node gets replaced with another, the client still accesses same IP.
2. "Corosync" is a Pacemaker service that uses heartbeat to track that nodes are available.
3. Upon nodes becoming unavailable, Corosync freezes resources and creates "quorum", and if majority of the cluster is alive and accessible, the majority sends STONITH to the dead part to make sure it's really dead and wouldn't create splitbrain.
4. "STONITH" *(shoot the other node in the head)* is a mechanism to physically disable the nodes, e.g. via IPMI and whatnot.
5. Data coherency between nodes is made sure via means unrelated to Pacemaker. One example is DRBD driver, which is a RAID1 over the network. So whenever node dies, the information is still available from the cluster.

2-nodes case worth mentioning separately. Here, the quorum would be tied, so in default configuration neither node can kill the other. One way to break the tie is by using third monitoring devices. Another one is by making nodes both send STONITH, but with different delay. So in 99% of cases one node will be alive and the other one dies. The 1% is a case where due to some perturbances the delays may match, making both nodes die.
