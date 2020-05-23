# NFS vs SMB vs SSHFS vs GlusterFS

* sshfs is simple setup, supports compression.
* GlusterFS is also simple to set up, supports compression. Separate kudos to GlusterFS for actually useful logs, unlike SMB and NFS. The bad thing though, OOTB sharing an arbitrary directory on your host by mounting it inside "shared volume" works read-only. Write attempts would throw `Stale file handle`. Idk if it can be worked around, after measuring performance I didn't care to debug it.
* Both NFS and SMB got me terrible experience. No compression support, complicated to set up, and logs are useless when something goes wrong *(yes, with verbose levels enabled)*.
* Bad for all four of them: neither is good at caching, trying to work with git remotely results in long hangs every time even though you'd expect client to have cached the files *(and it was tested on a machine with over than 100GB of RAM)*.

## Performance

Versions used for testing:

* Server: Archlinux as of 17.05.2020 *(so all sw is latest released as of the date)*
* Client: Ubuntu 18.04 *(so the client sw is old)*
* glusterfs is an exception: on Archlinux 7.4 version was used and on Ubuntu 7.6

For GlusterFS no cache was used simply because as of writing the words, `man mount.glusterfs` doesn't mention anything that seems interesting regarding these tests. I left out compression too because testing NFS shows quite good results even without compression.

Results of timing a python script in the shared dir, which basically loads lots of files inside the share. No local test in this table because script requires quite specific environment to work.

FS                                       | test 1             | test 2
---------------------------------------- | ------             | -----
sshfs with compression and cache options | 33 sec 014 ms      | 31sec 464 ms
nfs with cache *(fsc,nocto)*             | 29 sec 382 ms      | 4 sec 485 ms
GlusterFS with defaults                  | 37 sec 919 ms      | 42 sec 667 ms
samba with cache *(fsc,cache=loose)*     | 1 min 21 sec 49 ms | 1 min 19 sec 95 ms

Results of running `time git status` *(or actually `time git status && time git status`)* in the same shares *(after shares got remounted, so no cache remained from the prev. test)*. Worth noting, the share has lots of files permissions modified, i.e. there was a lot of content in the command output.

FS                                                        | test 1              | test 2
----------------------------------------                  | --------            | -----
sshfs with compression and cache options                  | 3 min 35 sec 78 ms  | 2 min 57 sec 79 ms
nfs with cache *(fsc,nocto)*                              | 3 min 44 sec 86 ms  | 2 min 03 sec 11 ms
GlusterFS with defaults                                   | 4 min 00 sec 79 ms  | 3 min 11 sec 04 ms
samba with cache *(fsc,cache=loose)*                      | 16 min 36 sec 68 ms | 15 min 08 sec 92 ms
Running locally after `echo 3 > /proc/sys/vm/drop_caches` | 1 sec 457 ms        | 0 sec 066 ms

So, what gives. SMB has so bad performance, that even if you sum up performance of SSHFS and NFS, SMB still gonna lose. Both SMB, SSHFS, and GlusterFS are bad at caching. And NFS has at least caching working and is the winner.

# NFS

## Setup

I got stuck for a looong time on fact that the mounted dir has read-only files, that is despite having `rw` permissions both in `mount` options and in server config, and despite the exported path on the server having `777` permissions. It could be a bug as well, with these useless logs you can basically for any problem just go read source code. But in the end I figured I had to add options `,all_squash,anonuid=1000,anongid=1000` to `exports` file.

The complicated thing is, NFS doesn't allow to just share a directory. You need to create a directory under root, mount the directory you wanted to share, and then share the directory you mounted it over. It goes like:

Suppose you want to share `/home/myuser`. First do on server:

1. Create `/export/myuser` dirs and do `chmod 0777 /export/ && chmod 0777 /export/myuser`
2. Run `sudo mount --bind /home/myuser /export/myuser`
3. Add to `/etc/exports` file:
    ```
    /export        myclient/24(rw,fsid=0,insecure,no_subtree_check,async,all_squash,anonuid=1000,anongid=1000)
    /export/myuser myclient/24(rw,nohide,insecure,no_subtree_check,async,all_squash,anonuid=1000,anongid=1000)

    ```
4. Run `sudo exportfs -a`
5. Start `sudo systemctl start nfs-server`

Then on client:

```
sudo mount -t nfs -o proto=tcp,fsc,nocto myserver:/ nfs-test/
```

# SMB

## Setup

I must admit, it is even worse to setup than NFS. Its logs were also unhelpful in debugging problems I stumbled upon.

While setup looks simple, I was trying for a long time to overcome `permission denied` while mounting the share. It's interesting that I could share a `/tmp` dir but couldn't a `/home/…` one. I hacked it around by using the common recommendation for NFS: I created a dir `/exports/foo` with `0777` permissions for both of them, and then `mount --bind`ed the target dir over there. And then I shared the `/exports/foo` instead of the one at `/home/…`

Afterwards I had a weird situation *(for both tmp and home shares)* where I could see and read files, they had correct permissions, and in various combinations of setup I tried they had the correct users too. But I never could change files, it was giving `permission denied` me. In the end I noted: if you mounted trough `sudo` you can change files as root but not as a usual user, despite files permissions having a common username/group. Anyway, this is the best I got with motivation at hand. This setup, that kind of works, is the following:

On the server `smb.conf` *(applying configuration requires `sudo systemctl restart smb nmb`)*:

```
[global]
   workgroup = WORKGROUP
   server string = Samba Server
   hosts allow = 192.168.1. 192.168.2. 127. 172.16.11.49
   log file = /var/log/samba/%m.log
   max log size = 50
   map to guest = Bad User
   dns proxy = no

[tmp]
   comment = Temporary file space
   path = /tmp
   read only = no
   writable = yes
   public = yes
   guest ok = yes

[export_foo]
   comment = foo
   path = /export/foo
   read only = no
   writable = yes
   public = yes
   force user = constantine
   guest ok = yes
```

Now you should see this share with `smbclient -L localhost -U%` *(but not with `smbtree`, I didn't explore why, perhaps something with "browseable" permissions)*

Client: mounting the share:

```
mount -t cifs -o rw,guest,fsc,cache=loose //172.16.23.156/export_foo foo/
```

# sshfs

I don't care enough to mention the server `ssh` config because it's simple, it's mostly defaults.

The sshfs call might be of interest though:

```
sshfs -o allow_other,kernel_cache,cache=yes,compression=yes constantine@myserver:/home/foo/bar mnt_dir
```

# GlusterFS

The only obstacle in configuration I stumbled upon is that it binds itself on the network address you mention in `…volume create…` command, so if you got multiple IPs and a client tries to connect by using a different one, it won't succeed.

Server:

```
mkdir -p /gfsvolume/gv0
gluster volume create my_vol transport tcp server_ip:/gfsvolume/gv0 force
gluster volume start my_vol
```

Client:

```
mount -t glusterfs server_ip:/my_vol /mnt/gfsvol
```
