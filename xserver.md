# DRI3

The infamous confusing `"GLX: Initialized %s GL provider for screen %d\n"` is at glxext.c of xserver source code. The DRI-version is coming from `__glXProviderStack` being a linked list filled through `GlxPushProvider()`. So far the only non-hardcoded provider I found is at glxmodule.c, line `provider = LoaderSymbol("__glXDRI2Provider");`.

Overall, it seems initializing a screen with DRI3 is not implemented even in slightest.

# tearing

Tearing prevention in xf86-video-ati is implemented with, quoting, `the hardware page flipping mechanism` since its begining.

# miscellaneous

CRTC can have more than one output. Acc. to docs it seems that a crtc determines width, height, and blank period.

# ATI Radeon DDX miscellaneous

`xf86CrtcPtr` have at least `xf86CrtcPtr::scrn` field, which allows getting screen configuration *(i.e. `xf86CrtcConfigPtr`)* through `XF86_CRTC_CONFIG_PTR()` macro.

## `drmmode_crtc_update_tear_free()`

There is `xf86CrtcConfigPtr::num_output` and `xf86CrtcConfigPtr::output[]::driver_private::tear_free`. Outputs are probably screens; each of them has `crtc` pointer. `drmmode_crtc_update_tear_free()` searches the one equal to the argument, and compares `tearfree` with 1 or 2 to figure if it needs to set `crtc->driver_private->tear_free` to true.

Apparently the `crtc : xf86CrtcPtr` arg is a product of driver and screen configuration. The function is trying to figure if the given `crtc` accord to an enabled output, and then if the output has tear_free enabled, enables it in the crtc as well.

## `drmmode_set_mode_major()`

It's an odd function. Major part of it is about searching for the `crtc` arg in various structs, and upon finding setting up something in those structs. It sets up x, y, rotation, dpms, hwcursor, anti-tearing. It also saves previous x, y, mode, and rotation in case configuration fails thoughout the function.

## `drmmode_crtc_scanout_update()`

Optionally creates and registers, then un-inits damage region, which I have no idea what.

All I found useful is that down the stack it calls `radeon_sync_scanout_pixmaps()`, and that `scanout_id` upon xoring with 1 points to the second pixmap used by tearfree. To be more specific, snip from the  `radeon_sync_scanout_pixmaps()`:


    DrawablePtr dst = &drmmode_crtc->scanout[scanout_id].pixmap->drawable;
    DrawablePtr src = &drmmode_crtc->scanout[scanout_id ^ 1].pixmap->drawable;

Apparently `scanout_id` is guaranteed to be 0 or 1.

# temporary resolution change for games

Resolution gets changed at `ProcVidModeSwitchToMode()` at `vidmode.c`. At the end, I presume, it cycles over existing modes, and compares to the requested. Given a match, it sets the new mode.

`SwitchMode()` calls `xf86SetSingleMode()` *(at least at one of the instances — but I assume there's only one)*.

# possible refactoring

* output should be stored at crtc *(instead both are at `xf86CrtcConfigPtr`, which leads to checks whether crtc is enabled, or owns the output.)*.
