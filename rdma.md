# General

Requires special support from apps. An app puts a message to a buffer, which is then directly read by network hw. Main point: OS not involved at all *(well, other than in setting up the communication channel)*.

* Infiniband: an RDMA technology. It is used for interconnected nodes, i.e. the computers are directly connected to each another. An OS sets up a communication channel, then gets out the way. From what I gather, an app puts a message to a buffer and issues a Write request. IB then transfers the buffer over the interconnect *(sequences on the way as needed)* to another node. Once the transfer is complete, the receiving app gets a notification.
  Also, a quote from Infiniband docs:
  > For both RDMA read and write, the remote side isn't aware that this operation being done (other than the preparation of the permissions and resources)
