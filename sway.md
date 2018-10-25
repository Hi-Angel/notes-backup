# general

The code seems to be badly designed, lots of global variables' usage.

Judging by having a pid in `bar_config`, it's rather a `bar_status`. Maybe worth renaming.

# rendering

wlroots manages cursor constraints; however it *does not* manage positions of surfaces. I.e. only compositor knows when a cursor crossed a surface, requiring to send a `wlr_seat_notify_pointer_enter` or `wlr_seat_notify_pointer_motion` to the client.

# IPC

i3 docs that sway implements: https://i3wm.org/docs/ipc.html
A library for i3: https://github.com/acrisci/i3ipc-python

# misc

Config entities implementation is in `sway/commands` dir.

`sway_view` whatever it is, can have a single container.

Some interesting structs:

    struct sway_root;
    struct sway_output;
    struct sway_workspace;
    struct sway_container;
    struct sway_transaction_instruction;
    struct wlr_box;

# commands

A stuff that can be bound to keypresses, came from i3.

# bars

Each one identified through IPC by `char *id`.

sway's `load_swaybars()` seems to rather "reload" them, because it deletes old ones, then invokes new bars. It's probably called so because functional of checking/deleting old ones was added later.

Part of bars' boot sequence is: `main` → `bar_setup` → `ipc_initialize`.

IPC note: in `invoke_swaybar()` sway makes a pipe, then forks a process for a bar. Then sway (i.e. the parent) checks whether there's `sizeof(size_t)` bytes sent by the child, in which case it's a length of an error, which sway then reads too.

It gets configs at `ipc_initialize`; specifically the line:

        char *res = ipc_single_command(bar->ipc_socketfd,
                IPC_GET_BAR_CONFIG, bar_id, &len);

Config parsing is at `ipc_parse_config()`

# reload config instead of restarting a bar

`cmd_reload()` does:

1. stores bars' ids
2. loads a config
3. calls some `ipc_event_workspace()`, which doesn't seem to do anything useful, just stores something into json hanging in the air *(an odd way of storing logs?)*.
4. terminates old bars, and invokes new ones, with `load_swaybars()`
5. for each bars' id that matches an old one calls `ipc_event_barconfig_update(bar)`

Apparently I need to remake `load_swaybars()` function. I gotta send `IPC_GET_BAR_CONFIG` to every bar from sway, over the IPC — can I?

# json

It is used in IPC.

# transparency/opacity

Opacity is a command. E.g. `bindsym $mod+f [<optional_criteria>] opacity 0.5` works. The command can be applied to specific windows through "criteria" *(optional)*.

`cmd_opacity()` receives an opacity command, then cycles over all containers on all outputs.

`output_damage_whole_container()` calculates some relation of current container to global position for given `sway_output`. Then scales with `scale_box`. Then gives out to `wlr_output_damage_add_box()` to (quote) "accumulate a damage and schedule frame event".

# questions

Can we not pad by 1px at `output_damage_whole_container`? See the comment, in the function, it sounds like they just want ceiling.

What's `focus_stack`?

# events

`window` when a window received input focus or when certain properties of the window have changed. There're bunch of properties, such as "fullscreen", "focus", "close", etc.

# input

## wlroots

`wlr_backend` abstracts outputs and input devices.

`wlr_backend::events::new_input` signals when a new input device was plugged in, with a reference to `wlr_input_device`. The later has a `type` field to show a pointer to which field is valid. E.g. `wlr_input_device::wlr_keyboard`.

`wlr_input_device::wlr_keyboard` provides its own events, like `key` and `keymap`. To convert its raw keys acc. to layout one needs to set up an xkbcommon context for it. [Here's an example][1].

`wlr_cursor` a helper to handle cursor painting, hw planes usage, output boundaries constraints… For handling hw cursors and output constraints it needs a `wlr_output_layout` to be bound.

`wl_seat` is a set of input devices. When a client binds to `wl_seat` a `wl_keyboard`, and `wl_keyboard::enter` is raised on their surface, it means the surface has keyboard focus. Alike for `wl_pointer` and `wl_pointer::enter`.

Per drewdevault:

> wlroots doesn’t render client surfaces for you, and doesn’t know where you put them. Once you figure out where they go, you need to notice when the `wlr_cursor` is moved over it and call `wlr_seat_pointer_notify_enter` with the pointer’s coordinates relative to the surface it entered, along with any appropriate motion or button events through the relevant `wlr_seat` functions. The client will also likely send you a cursor image to display - this is done with the `wlr_seat.events.request_set_cursor` event.
 > When you decide that a surface should receive keyboard focus, call `wlr_seat_keyboard_notify_enter`. `wlr_seat` will automatically handle removing focus from whatever had it last, and will also grab the keymap and send it to the client for you, assuming you configured it with `wlr_keyboard_set_keymap`… you did, right? `wlr_seat` also semi-transparently deals with grabs, the sort of situation where a client wants to keep keyboard focus for longer than it normally would, to deal with a context menu or something.

`xkb_keymap` holds something like possible layouts, e.g. `en,ru`.

`sway_seat->wlr_seat` has `wlr_keyboard`.

`struct wlr_input_device *wlr_device = keyboard->seat_device->input_device->wlr_device;`

## per-window layout impl.

Add it to a surface. Q: what to add?

## see also

https://drewdevault.com/2018/07/17/Input-handling-in-wlroots.html

[1]: https://github.com/swaywm/wlroots/blob/4984ea49eeaa292d66be9e535d93a4d8185f3e18/examples/simple.c#L114
