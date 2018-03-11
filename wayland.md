# bestiary

`request` is a compositor's method that a client might want to call.

`event` is something that happens with a compositor that a client sets a callback to.

`globals` are global resources, stuff like `wl_outputs` *(which displays are connected; also this specific global supplies `wl_registry`)*.

`wl_registry` is a list of available `globals`.

# misc

An extension is basically bunch of callbacks between a client and compositor. Details depends on extension.

Clients are usually using libwayland which to supply callbacks for certain events.

# references

http://www.jlekstrand.net/jason/projects/wayland/language-bindings-guide/ nice overview of how the xml sticks together with the final code.
http://sircmpwn.github.io/2017/06/10/Introduction-to-Wayland.html wayland from a POV of client.
