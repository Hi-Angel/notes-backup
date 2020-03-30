# jnlp files

You need to install java runtime and `icedtead-web`. Apparently, the latter can work with various runtimes, and various runtimes may coexist, so watch closely for using the exact java version you need. In my case I issued `sudo pacman -S java-runtime=8` and removed all other java versions *(though I tinkered before that quite a bit manually, so it's possible that something else might be needed)*. An IPMI I worked with did work with java 8.

`icedtead-web` provides `javaws`, and that it works can be checked by running `javaws -viewer`, it should launch a graphics interface.

`javaws` can be used to run jnlp files, but you probably would want browser to have them open automatically.
