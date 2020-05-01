# Using TCP/IP traffic shaping to achieve iSCSI service predictability

It seems published at 2010 year. Has references to a material of 2009, so looks truthful.

They talk about prioritizing iSCSI workloads. They say, there was no common way to do that while using FiberChannel, but since SANs started using TCP/IP for communication, TCP/IP can be prioritized.

The problem the paper is about: given a system providing virtual devices to consumers *(a SAN of comprising physical resources, and providing virtual disks)*. Now, one consumer may over-use IO thus worsening experience for other consumers for no reason. The paper outlines an algo to establish throttling to that IP using the `/proc…` and the like directories. There're also graphs "before" and "after".

Makes me wondering, haven't such algo been implemented in iscsi for example? I'd be surprised if there was no way to leverage existing Linux kernel algos for such usecase through some configuration.
