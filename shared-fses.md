# NFS vs SMB vs SSHFS

sshfs is simple to setup, and it supports compression while nfs and samba don't.

Both NFS and SMB got me terrible experience. No compression support, complicated to set up, and logs are useless when something goes wrong *(yes, with verbose levels enabled)*.

Bad for all three of them: neither seem to be good at caching, trying to work with git remotely results in long hangs every time even though you'd expect it to have cached the files *(and it was tested on a machine with over than 100GB of RAM)*.

## Performance

Results of timing a python script in the shared dir, which basically loads lots of files inside the share:

FS                                       | test 1  | test 2
---------------------------------------- | ------  | -----
sshfs with compression and cache options | 33 sec 014 ms  | 31sec 464 ms
nfs with cache *(fsc,nocto)*             | 29 sec 382 ms  | 4 sec 485 ms
samba with cache *(fsc,cache=loose)*     | 1 min 21 sec 49 ms | 1 min 19 sec 95 ms

Results of running `time git status` *(or actually `time git status && time git status`)* in the same shares *(after shares got remounted, so no cache remained from the prev. test)*. Worth noting, the share has lots of files permissions modified.

FS                                       | test 1   | test 2
---------------------------------------- | -------- | -----
sshfs with compression and cache options | 3 min 50 sec 72 ms  | 3 min 50 sec 72 ms
nfs with cache *(fsc,nocto)*             | 3 min 44 sec 86 ms  | 2 min 03 sec 11 ms
samba with cache *(fsc,cache=loose)*     | 16 min 36 sec 68 ms | 15 min 08 sec 92 ms
Running locally after `echo 3 > /proc/sys/vm/drop_caches`              | 1 sec 457 ms        | 0 sec 066 ms

So, what gives. SMB has so bad performance, that even if you sum up performance of SSHFS and NFS, SMB still gonna lose. Both SMB and SSHFS are bad at caching. And NFS has at least caching working and is the winner.

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
