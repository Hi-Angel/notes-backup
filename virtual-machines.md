# Making use of `docker export`ed files to run in a VM.

It's based on this article https://iximiuz.com/en/posts/from-docker-container-to-bootable-linux-disk-image/

First of all, the container needs to have kernel and a bootloader installed, here I assume it's done and the bootloader is Grub.

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
    sudo grub-install --target=i386-pc ${loop_path}                          # install bootloader
    udisksctl loop-delete -b ${loop_path}                                    # rootlessly remove loop
    qemu-system-x86_64 -drive file=bds.img,index=0,media=disk,format=raw     # start a VM
    ```
