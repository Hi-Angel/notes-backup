# Misc

* in the code, it seems "port" is a `route`. Whenever they print a port, they use `route::name` Not sure the reverse is true, but if you wanna change some logic for dealing with ports *(like handling of headphones plugging in/out in `pipewire-media-session`)*, look for routes.

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
