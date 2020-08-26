# Packaging

Involves running `fakeroot dpkg-deb --build .deb …` on the files. Minimal requirements seems to have a `DEBIAN/control` file. The `DEBIAN` dir may contain more files, specifically scripts running upon various events, but they're optional.

## Resolving file conflicts

Putting `dpkg-divert` call into `preinst` and `postrm` scripts seems to the way to deal wit it, but in reality that gives like a million of corner cases, such as installation, removal, upgrade, installation-fail *(and add the word "fail" to anything situation you can imagine)*, all of that may result in `dpkg-revert` failing. It's a pretty bad piece of software, it doesn't handle even such trivia as "clash of diversion of package foo by package foo", putting all of that burden on you.

Anyway, if you're still curious how to use it, do:

1. `dpkg-divert --add --rename --divert /etc/myconfig.old /etc/myconfig` to "divert" the `myconfig` into `myconfig.old`
2. `dpkg-divert --rename --remove /etc/myconfig` to remove the diversion.
