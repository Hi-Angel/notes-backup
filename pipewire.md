# Misc

* in the code, it seems "port" is a `route`. Whenever they print a port, they use `route::name` Not sure the reverse is true, but if you wanna change some logic for dealing with ports *(like handling of headphones plugging in/out in `pipewire-media-session`)*, look for routes.

# Terminology

* `sink` — speakers, headphones
* `source` — a microphone

# Modules

Modules loaded and unloaded with `pactl`. Worth noting that unloading a module by name seems not to work, only by using ID. This annoys while debugging, when you frequently load/unload, and each to you'd have to search ID. A workaround is to just use something like `pactl unload-module module-null-sink`, which will unload all null sink modules.

[Usage examples](https://gitlab.freedesktop.org/pipewire/pipewire/-/wikis/Virtual-Devices#create-a-source). Basically, everything is implemented via `module-null-sink`, which does different actions depending on its `media.class`. For example:

* Create a virtual sink: `pactl load-module module-null-sink media.class=Audio/Sink sink_name=my-sink channel_map=surround-51`
* Create a virtual source: `pactl load-module module-null-sink media.class=Audio/Source/Virtual sink_name=my-source channel_map=front-left,front-right`. To use it later you can link a real microphone `capture` to `input`s of the virtual micro, and then to actually listen to the sound use `capture`s of the virtual micro.

Later, ports of each modules can be connected with others or with applications by either using graphical frontend *(for example `qpwgraph`)*, or with `pw-link`. Use `pw-link -o` and `pw-link -i` to see available nodes.

# Researching issues

## No switch to headphones when plugged in

Dbg output of the pipewire-media-session:

took out headphones:
    * `handle_device` says: `available changed` prints for 2 mics and one headphones 0 -> 1
    * 1st match: for a `micro` and the `headphones`. Why headphones, what does that mean?
        * sound doesn't work at this point while I'm on a breakpoint there
        * then the loop finished
        * […] didn't check reconfigure_profile
    * 2nd match: for a `micro` and the `speakers`.
        * then the loop finished
        * sound started working only after I continued last breakpoint
        * […] didn't check reconfigure_profile

plugged in headphones:
    * `handle_device` says: `available changed` prints for 2 mics and one headphones 1 -> 0
    * 1st match: for a `micro` and the `speakers`.
        * […] didn't check sound
        * then the loop finished
        * stopped at reconfigure_profile, `restore = false`
    * no matches anymore

choose headphones manually:
    * NO `available changed`
    * 1st match: for a `micro` and the `headphones`.
        * `handle_route` says: `new active port found 'analog-output-headphones'[…]`, `restore route 4[…]`
        * then the loop finished
    * NO `available changed`
    * 2nd match: for a `micro` and the `headphones`.
        * `handle_route` said just: `port 'analog-output-headphones'`
        * then the loop finished

`parse_route` extracts a route from the argument.

The 1st `spa_list_for_each` loop seems to not have any influence over those devices because it only handles `SPA_PARAM_EnumRoute`, whereas the devices are `SPA_PARAM_Route`.

# TODO

* `handle_device` has two loops, which handle different types of devices *(`SPA_PARAM_EnumRoute` and `SPA_PARAM_Route`)*. They can be combined.
* can we make `p->id` an enum?
