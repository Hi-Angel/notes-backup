# general

The code seems to be badly designed, lots of global variables' usage.

Judging by having a pid in `bar_config`, it's rather a `bar_status`. Maybe worth renaming.

# misc

Config entities implementation is in `sway/commands` dir.

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

It's probably used in IPC. Better ask IRC for details.

# transparency/opacity

`cmd_opacity()` receives an opacity command, then cycles over all containers on all outputs.

`output_damage_whole_container()` calculates some relation of current container to global position for given `sway_output`. Then scales with `scale_box`. Then gives out to `wlr_output_damage_add_box()` to (quote) "accumulate a damage and schedule frame event".

# questions

Can we not pad by 1px at `output_damage_whole_container`? See the comment, in the function, it sounds like they just want ceiling.
