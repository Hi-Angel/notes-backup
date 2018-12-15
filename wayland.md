# general

Wayland works on "objects" that enclose methods. All requests are method invocations on them, and include an object ID. Each object implements an interface and the requests include an opcode that identifies method in the interface to invoke.

# bestiary

`request`: a message sent from a client to server.

`event`: a message sent from server to a client.

`wl_surface`: a rectangle that clients draws in. Essentially a window. Has events for whether it's visible.

`globals`: global resources, stuff like `wl_outputs` *(which displays are connected; also this specific global supplies `wl_registry`)*.

`wl_registry`: a list of available `globals`.

# misc

An extension is basically bunch of callbacks between a client and compositor. Details depends on extension.

Clients are usually using libwayland which to supply callbacks for certain events.

# references

http://www.jlekstrand.net/jason/projects/wayland/language-bindings-guide/ nice overview of how the xml sticks together with the final code.
http://sircmpwn.github.io/2017/06/10/Introduction-to-Wayland.html wayland from a POV of client.
