Terms: `target` is a server and `initiator` is a client.

# Connecting to iscsi target

For permanent connect there're settings to be set up. Otherwise, it can be done with command line utilties:

1. Fetch remote targets, so initiator knows about them: `iscsiadm -m discovery -t sendtargets -p IPAddr`
2. Connect to the target *(the targetname and ip:port you should see in the output of the discovery command)* `iscsiadm -m node --targetname iqn-of-target -p IP:port --login`
3. Find disks attached to the session: `iscsiadm -m session -P3` *(there's a lot of output, but you should see words "attaced scsi disk X")*

# Misc

* there are various… I think it's called drivers — apps to provide iscsi targets. One that you likely to find in tutorials online is called `tgt`. Another one is scst. They seem to conflict with each another if you use them simultaneously, but I'm not sure, problems I had could've been caused by something else.
* "iscsiadm: No portals found": I was getting this when trying to "discover" LUNs from a client. Debug logs of `iscsid` had nothing catchy. For me this was caused by lack of access in to LUN in target configuration for the client that ran discovery.

# Connecting a Ubuntu client
## *(optional)* create an internal iscsi iface

If you don't do this, the `default` iface will be used.

1. `iscsiadm -m iface -I iscsi01 --op=new`
2. Bind it to a network iface by its MAC: `iscsiadm -m iface -I iscsi01 --op=update -n iface.hwaddress -v 90:e2:ba:72:3a:74`
3. Am not sure on the relevance of this operation given the IP address is probably already assigned to the original iface, but: `iscsiadm -m iface -I iscsi01 --op=update -n iface.ipaddress -v 10.10.10.30**

## Making the connection

**Note**: omit `-I iscsi01` option if you skipped previous section.

1. Server: create an iSCSI lun and give it access to the client by its IQN that can be found at `/etc/iscsi/initiatorname.iscsi`.
2. Client: discover IQNs available on the target: `iscsiadm -m discovery -I iscsi01 --op=new --op=del --type sendtargets --portal 10.10.10.26`. We say `iqn.myiqn` further to refer to the IQN you got here.
3. Client: log in to bring the target devices in `iscsiadm -m node --targetname iqn.myiqn --login -I iscsi01`
4. Client: now, list the devices `iscsiadm -m session -P3`. There will be a text `Attached SCSI devices:` and then some information including the block-device names shall follow.
5. *(optional)*: client: if you have multipath, then it should've automatically grouped the devices mentioned under `iscsiadm -m session -P3`, and then `multipath -ll` should show what `/dev/mapper/*` devices were created as result.

# Connecting a Windows client with older BAUM NAS

1. Server: create a pool, volume.
2. Server: go to `Protocols → iSCSI`, enable the service, create a LUN for the volume.
3. Create a client with IQN
   * Client: open `Management panel → Initiator settings`, and click `Configuration` tab. There should be a field `initiator name` with IQN of the machine.
   * Server, go to `Access → Clients and Groups`, create a client, and as an IQN enter the one we got from the Windows machine.
4. Server: bind iscsi service to the IP addresses you gonna connect through.
5. Server: bind the client created to the LUNs.
6. Client, in the `Initiator settings`, go to tab `Endpoints`, enter the server IP addresses and press <kbd>Enter</kbd>. That should open a prompt with IQNs available on the server, choose one, press ok.
7. *(optional)* Client: open `Computer management → Disks management` *(or Run `diskmgmt.msc`)*, scroll at the lower half-screen the list of disks to the bottom, and find the new disk that should've appeared. Right-click its icon on the left, and press `Initialize`.
