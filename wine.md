# Debugging

It is possible to create a build of wine, and then to run wine right from the build directory. For example, `wine-git` from AUR creates two wine directories `build-32` and `build-64`. It is then possible to enter them, and use `make` command to rebuild; and use full path to `wine` symlink inside that build directory to run wine from there.

# Misc

* **running WINE from bulid dir**: it will provide a `./wine → tools/winewrapper` symlink that is a shell-script wrapper setting up the necessary env. variables and running wine. In case you really want it, you may do it all manually with a `LD_LIBRARY_PATH=libs/wine WINELOADER=loader/wine loader/wine /tmp/my.exe`
