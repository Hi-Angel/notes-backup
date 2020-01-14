# Misc

## Cisco Anyconnect

Downloadable from Cisco site for the 3 major platforms, but unfortunately only if you're 1) registered, and 2) have some contract with Cisco.

Much easier way is to install and use openvpn

## Openvpn

Once installed, a command to connect to a vpn is:

```
sudo openconnect -s /etc/vpnc/vpnc-script vpn-server --user=user-name
```

Things to consider:

1. It works well with 2-factor authorization. After entering the password, it will ask you for the code that was sent to your phone.
2. username may or may not include email address. Try both if unsure.
3. No need to create `tunX` interface, as some tutorials on the internet show. The `openconnect` will do that if needed.
