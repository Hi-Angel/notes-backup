# Making use of `docker export`ed files to run in a VM.

It's based on this article https://iximiuz.com/en/posts/from-docker-container-to-bootable-linux-disk-image/

1. Create the image `truncate 5G testfile`
2. Create a partitions with a script
    ```
    sfdisk testfile <<EOF
    label: dos
    unit: sectors

    partition1: start=2048, type=83, bootable
    EOF
    ```
3. In short:
    ```
    loop_path=$(udisksctl loop-setup -f ./bds.img | grep -oP "/dev/loop\d+") # rootlessly setup loop0
    sudo mkfs.xfs ${loop_path}p1                                             # create a FS
    mount_dir=$(udisksctl mount -b ${loop_path}p1 | grep -oP " at \K/.+")    # rootlessly mount the dir
    sudo tar -xzvf flash.tar.gz -C  ${mount_dir}                             # unpack the OS
    udisksctl unmount -b ${loop_path}p1                                      # rootlessly unmount
    …                          # install bootloader, see next paragraph
    udisksctl loop-delete -b ${loop_path}                                    # rootlessly remove loop
    qemu-system-x86_64 -drive file=bds.img,index=0,media=disk,format=raw     # start a VM
    ```

## Installing bootloader

While `grub` might seem like obvious choice, but in reality the `grub-install` is a terrible command *(detailed elsewhere in notes)*, and is a terrible choice for a VM. What worked for me was `syslinux` bootloader with its `extlinux --install` command. Note: while `extlinux` manual says it works with ext2, ext3 — this is not true, it works with many modern FSes as well. I used it with XFS in particular.

Example of using it as part of actions above:

```
sudo extlinux --install ./mnt
sudo mkdir mnt/boot/syslinux/
cat <<EOF | sudo tee mnt/boot/syslinux/syslinux.cfg
DEFAULT linux
  SAY Now booting the kernel from SYSLINUX...
 LABEL linux
  KERNEL /vmlinuz
  APPEND ro root=/dev/sda1 initrd=/initrd.img
EOF
```

# QEMU

## Misc

* Network: do not use `bridge` in production as it makes NIC go to promiscuous mode, and then kernel sorts out the packets, which results in lots of copying and CPU usage. Instead either use `SR-IOV` *(NIC-dependent mode which requires the NIC to support it)* or `macvtap` *(it's about assigning multiple MAC addresses to a NIC and then using unicast filtering)*

# Hypervisors

## Proxmox

Open-source solution, allows managing QEMU/KVM and LXC. There's a paid version, but the free one has no limitations in the feature-set. Similarly to ESXi, it allows having multiple hosts/nodes and migrating VMs between them *(basically, both vCenter and ESXi are combined into one product)*.

## ESXi

VMWare proprietary solution. **ESXi** represents a single host/node that manages its own VMs. Up the hierarchy is a **vCenter**. It's a server that manages multiple ESXi hosts, and allows e.g. to migrate VMs between them. There's also **vSphere**, which is a GUI that combines both **ESXi** and **vCenter**.

VMWare provides a free version, but with limitations:

* No more than 8 cores for a VM
* **vCenter** management not allowed *(it will refuse to connect a "free" **ESXi**)*
* No vStorage API *(it's a backup API. Note that this is different from a ESX API for managing VMs)*
* All accessible APIs are read-only, i.e. modifying operations are not allowed. *(Note: this one is taken from many forum posts. But apparently VMWare does not provide official documentation enlisting the limitations, so 🤷‍♂️)*

ESXi is based on VMkernel which is a proprietary kernel developed by VMWare.
