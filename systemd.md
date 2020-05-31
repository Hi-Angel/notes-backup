# systemd-networkd

Allows to configure network. ATM doesn't seem to be integrated with NetworkManager, so not for desktop use. But for servers or embedded systems works ok.

## File example

Note: it should end with `.network` postfix, otherwise networkd gonna just ignore the file.

```
[Match]
Name=eth0

[Network]
Address=192.168.0.1/24

[Route]
Gateway=192.168.0.0/24
```
