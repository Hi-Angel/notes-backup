Has same CLI as Docker, but is better in terms of security *(easier audit, and there's also no services running as root)*, and no need to run a service for it to work. It is completely compatible with Docker.

Due to its similarity to `docker`, most of the stuff mentioned in `docker.md` applied to `podman` as well, so I see no need to copy it here.

# Initial setup

1. Create a `/etc/subgid` and `/etc/subuid` files with the following line *(substitute username with your user name)*: `username:100000:65536`. The 2nd column seems to be arbitrary, but tutorials prefer for it to start at least with 100k. For 3rd column, a 2¹⁶ is usually recommended.
2. Run `podman system migrate`
3. Create a `~/.config/containers/containers.conf` file *(to avoid problems with readline hotkey C-p)* with
    ```
    [engine]
    detach_keys=""
    ```

# Examples

* Run `ubuntu:18.04` with the directory `foo` mounted inside the container
    ```
    podman run --rm -v ~/Projects/foo:/home/user/Projects/foo -it ubuntu:18.04
    ```
* Run `bash` inside `img` instead of some default script that the author has set, achieved by clearing the "entrypoint":
    ```
    podman run --rm --entrypoint= -it img /bin/bash
    ```

    alternatively, launch a container as usual and then run separately `podman exec -it container_name /bin/bash`.

* Commit currently running `foo` as `foo`
    ```
    podman commit $(podman ps | perl -lane 'print @F[-1] if /foo/') foo
    ```

# Caveats

* Fedora has some lame setup where volume mounts won't work with podman *(specifically, you'll get "Permission denied" on accessing the mount)*. There are two ways to fix that:
  1. Better one: have a `SELINUX=permissive` line in the `/etc/selinux/config` file.
  2. Worse one: add a `:z` to the end of a mount, i.e. change `-v /foo:/bar` to `-v /foo:/bar:z`. It sometimes works, sometimes doesn't. `setctl` call on fails will fail. Also from what I've read, it seems might mess up some user ids inside the container, so that's nothing more than a hack. Setting `selinux` to `permissive` *(or even disabling selinux whatsoever)* is a better way to deal with this.
