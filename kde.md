# missing icons in Qt apps

Many DEs would require setting `QT_QPA_PLATFORMTHEME=qt5ct` *(and installing/using that qt5ct, that's a GUI configurator)*.

# development of arbitrary KDE apps

Some KDE apps and libraries are interdependent on upstream versions of each other. This means, to develop e.g. KMail you'd have to get the upstream version of it. There're multiple way to deal with it.

## kdesrc-build

ATM it's implemented terribly: you need to actually "install" stuff somewhere, and then `export` a bunch of stuff. You can't just do "make run kmail", thus making it to use everything you just built.

"Building the world" happens through `kdesrc-build`, see examples for kmail development here *(including the exports)*: https://community.kde.org/KDE_PIM/Development/Start#Compiling

`kdesrc-build` is probably bad in handling dependencies, as in my limited experience it haven't figured that kmail depends on upstream version of QtWebEngine, and I've been told on IRC that one indeed have to get upstream QtWebEngine some way around `kdesrc-build` *(e.g. as a package)*. In addition, upon asked to build kmail it went building 1xx apps, most of which have nothing to do with kmail; and worse of all, it failed to build 5x of them. So yea, it's definitely bad.

## docker or a virtualbox of `KDE Neon Developer Edition Unstable`

It's a distro with all that latest stuff. A docker image specifically for development PIM stuff *(which is the one most often requiring upstream versions)* can be found here https://community.kde.org/KDE_PIM/Docker
