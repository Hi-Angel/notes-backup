# Debugging

## Env. variables

The list is here https://docs.mesa3d.org/envvars.html *(and somewhere in Mesa sources I presume)*. Some are I've used or found otherwise useful.

* `LIVVA_DRIVER_NAME=foo` overrides vaapi default driver. Here, given foo, the driver file would be `foo_drv_video.so`
* `MESA_LOADER_DRIVER_OVERRIDE=foo` overrides a graphics driver binary. Given `foo`, the driver file would be `foo_dri.so`
* `LD_LIBRARY_PATH=/full/path/to/mesa-repo-root-dir/lib LIBGL_DRIVERS_PATH=/full/path/to/mesa-repo-root-dir/lib/gallium app` an outdated example of running an app with the built Mesa instead of the system one. "Outdated" because the paths are for pre-meson Mesa. Nowadays the paths are changed, gotta test it.
