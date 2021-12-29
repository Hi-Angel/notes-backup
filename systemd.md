# systemd-networkd

Allows to configure network. ATM doesn't seem to be integrated with NetworkManager, so not for desktop use. But for servers or embedded systems works ok. Configs are to be placed at `/etc/systemd/network`

## Config example

```
[Match]
Name=eth0

[Network]
Address=192.168.0.1/24

[Route]
Gateway=192.168.0.0/24
```

## Caveats

* Config should end with `.network` postfix, otherwise networkd gonna just ignore the file.
* it randomly *(what seems like it)* applies or not new settings on restart. That looks like a bug.

# Default target

Loosely saying, it's a target to run when system booted. Usually it is `graphical.target`. Can be queried with `systemctl get-default`.

# Debugging

* listing dependencies: `systemctl list-dependencies [optionally.target]`. Won't show services that weren't loaded into memory. Perhaps `systemd-analyze dump` might be more suitable for that. The `list-dependencies` may be useful for debugging errors like `foo.service: Job foo.service/start failed with result 'dependency'`, it's gonna show the service that failed.
