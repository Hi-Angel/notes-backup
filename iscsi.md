# Connecting to iscsi target

For permanent connect there're settings to be set up. Otherwise, it can be done with command line utilties:

1. Fetch remote targets, so initiator knows about them: `iscsiadm -m discovery -t sendtargets -p IPAddr`
2. Connect to the target *(the targetname and ip:port you should see in the output of the discovery command)* `iscsiadm -m node --targetname iqn-of-target -p IP:port --login`
3. Find disks attached to the session: `iscsiadm -m session -P3` *(there's a lot of output, but you should see words "attaced scsi disk X")*

# Misc

* there are various… I think it's called drivers — apps to provide iscsi targets. One that you likely to find in tutorials online is called `tgt`. Another one is scst. They seem to conflict with each another if you use them simultaneously, but I'm not sure, problems I had could've been caused by something else.
* "iscsiadm: No portals found": I was getting this when trying to "discover" LUNs from a client. Debug logs of `iscsid` had nothing catchy. For me this was caused by lack of access in to LUN in target configuration for the client that ran discovery.
