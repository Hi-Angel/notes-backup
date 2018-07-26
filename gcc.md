# Misc

## Building

Needs to be compiled in separate dir, i.e. `mkdir build`, `cd build`, `../configure …`, `make`.

Using CFLAGS, CXXFLAGS have been resulting in stage-comparision failure. What have worked though, is using `BOOT_CFLAGS` instead.
