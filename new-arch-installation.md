* format:
 * GPT: `mkfs.fat -F32` for 512M efi partition, mark it as `esp` *(`EFI system` in fdisk)*, `/` as btrfs.
 * MBR: format ext4 512Mb for `/boot`, and btrfs `/` for the rest.
* Mount the 2 partitions; enable zstd compression on btrfs.
* install with `pactrap` the base packages *(see [this page](https://wiki.archlinux.org/title/Installation_guide), basically just kernel)*
* use `arch-chroot` to enter the chroot
* Edit `/etc/fstab` for UUIDs and compression *(it's generated these days, see [this page](https://wiki.archlinux.org/title/Installation_guide))*
* install bootloader, but **not systemd-boot**. In my experience it's not detected by systems, better go with the old pal Grub.
* `pacman -S --needed plasma i3 xorg-server git wget gdb konsole sudo pipewire-pulse feh picom hunspell-{ru,en_us} man mold moreutils ttf-ubuntu-font-family xorg-xinput spectacle noto-fonts-emoji gnome-keyring libsecret seahorse libnma cmake extra-cmake-modules man-pages cups system-config-printer libreoffice-fresh meson base-devel libaio perf zsh-completions dnsutils gnome dbus-broker`
* `systemctl enable cups.socket NetworkManager dbus-broker && systemctl --global enable dbus-broker`
* create a user, set a password. As the user account:
    * copy my configs/dotfiles
    * `gio mime x-scheme-handler/http org.qutebrowser.qutebrowser.desktop && gio mime x-scheme-handler/https org.qutebrowser.qutebrowser.desktop`
    * `systemctl enable --user pipewire.socket`
* edit FLAGS in `/etc/makepkg.conf` to `-march=native -O3 -pipe -fmerge-all-constants -flto`
* reboot
* log into Gnome and launch some app that requires storing a password to keyring and store something *(presumably, a `secret-tool store --label="MyApp" myapp myusername` may work)*. This is needed for keyring to get properly initialized, otherwise it won't work in other DEs.
