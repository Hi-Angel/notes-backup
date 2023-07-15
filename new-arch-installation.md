* format:
 * GPT: `mkfs.fat -F32` for 512M efi partition, mark it as `esp` *(`EFI system` in fdisk)*, `/` as btrfs.
 * MBR: format ext4 512Mb for `/boot`, and btrfs `/` for the rest.
* Mount the 2 partitions; enable zstd compression on btrfs.
* use `arch-chroot` to enter the chroot
* Edit `/etc/fstab` for UUIDs and compression *(it's generated these days, see [this page](https://wiki.archlinux.org/title/Installation_guide))*
* install with `pactrap` the base packages *(see the mentioned page, basically just kernel)* and `--needed plasma i3 xorg-server git wget gdb konsole sudo pipewire-pulse feh picom hunspell-{ru,en_us} man mold moreutils ttf-ubuntu-font-family xorg-xinput spectacle noto-fonts-emoji gnome-keyring libsecret seahorse libnma cmake extra-cmake-modules man-pages cups system-config-printer libreoffice-fresh meson base-devel`
* install bootloader, but **not systemd-boot**. In my experience it's not detected by systems, better go with the old pal Grub.
* `gio mime x-scheme-handler/http org.qutebrowser.qutebrowser.desktop && gio mime x-scheme-handler/https org.qutebrowser.qutebrowser.desktop`
* edit FLAGS in `/etc/makepkg.conf` to `-march=native -O3 -pipe -fmerge-all-constants -flto`
* copy i3 config
* reboot
