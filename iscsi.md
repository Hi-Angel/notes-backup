# Connecting to iscsi target

For permanent connect there're settings to be set up. Otherwise, it can be done with command line utilties:

1. Fetch remote targets, so initiator knows about them: `iscsiadm -m discovery -t sendtargets -p IPAddr`
2. Connect to the target *(the targetname and ip:port you should see in the output of the discovery command)* `iscsiadm -m node --targetname iqn-of-target -p IP:port --login`
3. Find disks attached to the session: `iscsiadm -m session -P3` *(there's a lot of output, but you should see words "attaced scsi disk X")*

# Misc

* there are various… I think it's called drivers — apps to provide iscsi targets. One that you likely to find in tutorials online is called `tgt`. Another one is scst. They seem to conflict with each another if you use them simultaneously, but I'm not sure, problems I had could've been caused by something else.
* "iscsiadm: No portals found": I was getting this when trying to "discover" LUNs from a client. Debug logs of `iscsid` had nothing catchy. For me this was caused by lack of access in to LUN in target configuration for the client that ran discovery.

# Connecting a Windows client with BAUM BDS

1. Create a pool, volume.
2. Go to `Protocols → iSCSI`, enable the service, create a LUN for the volume.
3. Create a client with IQN
   * on Windows machine, open `Management panel → Initiator settings`, and click `Configuration` tab. There should be a field `initiator name` with IQN of the machine.
   * on the server, go to `Access → Clients and Groups`, create a client, and as an IQN enter the one we got from the Windows machine.
4. In the server, go to `Protocols → iSCSI`, open the drop-down menu `iSCSI Target`, then you should see 2 IQNs. They will be used in the next step.
5. On the Windows client, in the `Initiator settings`, go to tab `Endpoints`, and enter previously acquired IQNs into the text field "Object" and press `Quick connection` button. Do that for both IQNs separately.
6. *(optional)* open `Computer management → Disks management`, scroll at the lower half-screen the list of disks to the bottom, and find the new disk that should've appeared. Right-click its icon on the left, and press `Initialize`.
