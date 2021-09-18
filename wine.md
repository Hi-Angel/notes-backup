# Debugging

It is possible to create a build of wine, and then to run wine right from the build directory. For example, `wine-git` from AUR creates two wine directories `build-32` and `build-64`. It is then possible to enter them, and use `make` command to rebuild; and use full path to `wine` symlink inside that build directory to run wine from there.
