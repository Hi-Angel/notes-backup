# Misc

* server: `freerdp-shadow-cli`, client: `xfreerdp`.
* channels: various communication means between server and client. There are `static` *(created at connection time)* and `dynamic` *(may be created at any time)* channels. "dynamic channels" are transported over DRDYNVC, which is a static channel specialised for that purpose.
    * `echo` channel: sends message to client and then expects client to send it back. There's an echo protocol for that.
* verbose mode: setting `WLOG_LEVEL=DEBUG`
* [good article on the basics](https://www.cyberark.com/resources/threat-research-blog/explain-like-i-m-5-remote-desktop-protocol-rdp)

# Research

A channel is being loaded after user\domain was entered upon connection, from `drdynvc_virtual_channel_event_connected → dvcman_load_addin → foo_DVCPluginEntry` *(where `foo` is a channel/plugin name)*.
