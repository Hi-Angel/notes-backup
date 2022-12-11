# Config

* configs are written under `/etc/dracut.conf.d/foo.conf` files. It is preferred over writing directly to `/etc/dracut.conf` as this one is owned by a package, whose maintainers may want to update the file. So if you change it, you'll have to manually watch if there's any new modifications you'd want to have.
* `omit_dracutmodules`: dracut modules usually group files to include from the file system. This option allows to remove some from the initramfs, however, it is interesting that there are useless modules you wouldn't want to actually exclude:
    * the `i18n` localization module *(who needs localization in initramfs 🤷‍♂️)* is required by some systemd service that gonna start throwing errors *(though in my experience that didn't visibly influence the boot, besides showing the error for a moment)*
    * the network-related modules you'd think is a burden unless someone wants to boot through a network. Yet modules like `network-manager ifcfg kernel-network-modules` and something else on Fedora are all required for properly showing boot progress bar *(basically, for proper plymouth)*.
