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
* it does seem to apply config on restart. So far when it didn't, it were some problems with the config, like the missing extension as in prev. example.
