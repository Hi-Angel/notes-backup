# Misc

`firewalld` works nicely as a replacement to ufw, but needs some tinkering to make it log dropped packets, etc, into `journalctl`. Basically, this requires 1. setting `LogDenied=all`, and 2. adding rsyslog rule to log entries matching `_DROP` and `_REJECT`. Details: https://unix.stackexchange.com/a/503267/59928

A command adding a rule to allow multicast: 

```
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" destination address="224.0.0.1" protocol value="2" accept' --permanent
```
