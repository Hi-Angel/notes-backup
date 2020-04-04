# raid

Generally a data broken into multiple disks. Details depends on the type.

raid 0 — aka "performance at all costs". Every data piece is divided by number of disks. Shall a single drive fail, it gonna take down all data. Used when you don't care of data loss (e.g. caching). Total usable space is the sum of all drives.

raid 1 — aka "mirroring", the data is copied to all present disks. Depending on implementation reads can be better by reading data pieces from different disks. Total usable space is the size of the smallest disk.

raid 1e — performance similar to raid 1. The data is broken into blocks, and interleaved among several disks. A number of copies of each data block depends on configuration, but at least 2.

raid 10 — combination of 0 and 1. Overall, not worth remembering since it's an increased
complexity for vague means.

raid 01 — bad modification of raid 10.

raid 5 — performance of all drives. Data is stored in blocks to every drive, integrity is achieved by storing for every block [a xored value][2] which they call "parity". Redundancy equals to a single drive. Note: it allows for loss of only single drive, and because the following restoring of the data gonna cause an intensive IO, it can lead to a loss of another drive, which in turn unrecoverable.

Worth noting raid 5 with two disks vs raid 1: raid 5 allows for easy adding a new disk. Otherwise they look similar since "a xor id = a".

Also, instead of XOR could be used [erasure coding][3]. It is more reliable, and allows quicker data reconstruction.

raid 6 — like raid 5, but stores two "parity blocks". Allows for loss of two disks.

raid 50, 60 — I decided to gloss over, [see here][1].

# RAID software

* `mdadm` classic app for creating a RAID. Simple and mature.
* `lvmraid` per comparison on SE, it seems to be using md-raid under the cover. However as of 2015 *(date of the answer)* they say, its utilities are not as mature.
* `zfs` has embedded means for creating a RAID. Probably not as performant as mdadm due to CoW though, but it has its features.
* `btrfs` ditto as ZFS.

# LVM

Logical Volume Management — allows for representing multiple disks as a single one, an online modifications to partitions, etc. Basically, it's just an added easy-of-use and flexibility.

Layers used in order from the most primitive:

1. **physical volumes**, prefix `pv...` — block devices used for higher level.
2. **volume groups**, prefix `gv...` — storage pools of **physical volumes** that abstract characteristics of underlying devices.
3. **logical volumes**, prefix `lv...` or `lvm...` — slices of a **volume group**, it's like partitions.

# ZFS

There's a separate file `zfs.md` on ZFS

# BTRFS

* Creating a RAID: `mkfs.btrfs -m raid5 -d raid5 /dev/sd{g,h,i,j,k,m,o,p,r,v}1 -f`. Then to start using it you have to run `mount` command on *any* of the devices, like this `mount -t btrfs /dev/sdg1 /tmp/btrfs-mnt`.
    You may well ask: but can BTRFS RAID provide a raw block device that can be later formatted as `ext4` or whatever. The answer is "No", however you can emulate it by creating a file of required size, and creating a loopback device over it.
* Destroy the RAID: apparently, there's no dedicated command to it. After talking on IRC, the way to get rid of above RAID would be `wipefs -a /dev/sd{g,h,i,j,k,m,o,p,r,v}1`. I was told that btrfs rescan running might be needed, but as of kernel 5.6 I didn't need it, i.e. `btrfs fi show` didn't show the devices anymore.

# SMART tools

Report a disk health.

## Failures

Turns out, ongoing failure of a drive can't be determined for sure just by looking at stats. Instead, people using smart tools take advantage of statics: given an attribute, how often with its value greater than zero a drive fails in future or remains operational.

[Per this article](https://www.backblaze.com/blog/what-smart-stats-indicate-hard-drive-failures/)

For attribute 184 "End to End Errors" bigger than zero, 1.5% of drives fail and just 0.1% operational drives report it.

Given attributes 5, 187, 188, 197, 198: if two or more of them simultaneously greater than zero, this almost always means ongoing fail. There's other fun statics about their relationship over at the article linked.

189 "High Fly Writes": this one steadily increasing over months means nothing. However having it increased by a few dozens for a period of just one week means you can start worrying.

# CEPH

Cluster based storage system. Stores data across the network. Also employs erasure coding.

Allows making snapshots.

`OSD` is like a storage device.
`Object` is a file stored by clients (given "file" makes sense for a client — e.g. there's a support to show whole cluster as a block device). An object has identifier, payload, and a number of attributes.

## CRUSH acc. to a colleague + paper.

There's a tree with weights assigned.

## CRUSH in CEPH

Contrary to block-based HDDs, CEPH relies on object-based storage devices (OSD), where objects are variable-sized/named. A file to be stored is broken to bunch of named objects.

The algo determining storage targets, given placement rules, cluster map, and x.

Devices — storage devices. Gets assigned weights to control relative amount of data they responsible for storing. There's something looking like recommendation to derive the weight from the disk's bandwidth, capacity, and the likes. Have a known fixed type.

Buckets — any number of devices or buckets (a homogenous set). Their weight is the sum of weights of devices. Has type field to distinguish bucket-of-buckets from bucket-of-devices.

There's a storage graph where devices are always leaves, and buckets are (always?) internal nodes.

Buckets can be composed arbitrarily to construct a hierarchy representing available storage. For example, one might create a cluster map with “shelf” buckets at the lowest level to represent sets of identical devices as they are installed, and then combine shelves into “cabinet” buckets to group together shelves that are installed in the same rack.

### paper notation

object — a data block.
bucket — a set of either devices or another buckets.
x — object name(?) or other object identifier.
n — number
t — a node type.
ivec — a set of buckets, taken the cluster tree.
take(a) — takes a from the cluster tree, then puts a into ivec.
select(n,t) — searches&selects for n distinct items of type t within ivec.
r — index of the replica being chosen.
c(r,x) — defined for every node type, descends to it
emit() — simply returns the result (the veci)

### Chapter "Replica Placement"

Tells a story that by assigning weights to devices&buckets one can configure data placement. Then goes describing algorithm which simply chooses nodes with alike but descending semantic meaning, like "root", "row", "cabinet", all the way down to one disk inside every cabinet.

It notes that `select()` may reject an item if it's already chosen, or failing, or overloaded. Failing device is marked as such, but left in the map to avoid unnecessary (why?) data shift. Overloaded devices get pseudo-randomly rejected with an established probability.

For failed&overloaded devices CRUSH redistributes objects along the cluster by retarting recursion of `select()` in the root.

If `select()` gets a collision, an alternate r' is used at inner levels (why "inner levels"?).

"Replica Ranks" subchapter notes that parity and erasure coding schemes have to be handled specially, and instead of choosing "first n available targets" it chooses "first n multiplied by number of fails".

### Chapter "Map changes and data movement"

Notes that though failed & overloaded devices can be left in place, but the situation is more complex when storage resources removed or added. New devices have weights, and acc. to the weights data starts relocating (does it cache prev. map to figure where data was before?).

It also describes some statistic properties of the algo.

### Chapter "Bucket types"

Describes bucket types, and algo of the function `c()` for them. There is a trick — 3.4.4 describes a bucket type that works with weights. It is not obvious, but e.g. a type from 3.4.1 does not work with weights, but rather there for grouping similar devices i.e. that would have same weight, like devices of same capacity, speed, placed in the same cabinet.

So in fact this is the chapter that describes how do weights work in CRUSH.

### Questions

1. How are weights used?
2. Given a filename, how does crash determine where blocks are stored (physical address?), and in what order?

### sum up

I could try implementing the graph of devices (tho not sure what to do with weights and physical storage), and see if implementation gonna make any sense. In fact it's the best I can do now.

I could also ask at #ceph channel the questions, though if BUAM has a guy with the knowledge, it might be more productive.

### Notes

Given CEPH is using CRUSH with file systems, it could simply store a file block with the file name+block id.

CRUSH algo is like a trained ANN, except that we never know what changing a weight in an ANN might end up in.

"pseudo-random reproducible function" being a hash of values.

# References

1. https://serverfault.com/questions/339128/what-are-the-different-widely-used-raid-levels-and-when-should-i-consider-them
2. https://accu.org/index.php/journals/1915 XOR used for data recovery if a single disk fails by having stored all disks xored to another one.
3. http://www.computerweekly.com/feature/Erasure-coding-versus-RAID-as-a-data-protection-method erasure coding in RAID
