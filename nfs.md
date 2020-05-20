# nfs vs smb vs sshfs

SMB I didn't try at this point, can't say much here. Doesn't support compression.

sshfs is very simple to setup, and it also supports compression while nfs and samba don't. So if you work with a project over remote directory, sshfs may be a better fit *(but I didn't actually measure)*.

With NFS I've got so far bad experience. Doesn't support compression, complicated to set up, its logs are useless when something goes wrong. And I got stuck for a looong time on fact that the mounted dir has read-only files, that is despite having `rw` permissions both in `mount` options and in server config, and despite the exported path on the server having `777` permissions. It could be a bug as well, with these useless logs you can basically for any problem just go read source code. But in the end I figured I had to add options `,all_squash,anonuid=1000,anongid=1000` to `exports` file.

Bad for both sshfs and NFS: neither seem to be good at caching, trying to work with git remotely result in long hangs every time even though you'd expect it to have cached the files *(and it was tested on a machine with over than 100GB of RAM)*.

Speed of running a noop `make clean` *(i.e. when there was no actually object to clean)* of an arbitrary project:

* sshfs with compression and cache options: 1m32.306s
* nfs: 0m42.542s

# Setup

The complicated thing is, NFS doesn't allow to just share a directory. You need to create a directory under root, mount the directory you wanted to share, and then share the directory you mounted it over. It goes like:

Suppose you want to share `/home/myuser`. First do on server:

1. Create `/export/myuser` dirs and run `sudo mount --bind /home/myuser /export/myuser`
2. Add to `/etc/exports` file:
    ```
    /export        myclient/24(rw,fsid=0,insecure,no_subtree_check,async,all_squash,anonuid=1000,anongid=1000)
    /export/myuser myclient/24(rw,nohide,insecure,no_subtree_check,async,all_squash,anonuid=1000,anongid=1000)

    ```
3. Run `sudo exportfs -a`
4. Start `sudo systemctl start nfs-server`

Then on client:

```
sudo mount -t nfs -o proto=tcp myserver:/ nfs-test/
```
