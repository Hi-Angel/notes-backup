# jnlp files

They are XML files that typically point out the URL to download the actual `.jar` file from and a some parameters *(e.g. an auth cookie)*. The `jar` file itself then may be signed with some old algo that is *(hopefully)* still supported but was disabled. In this case you'd get an error:

```
Application Error: Cannot grant permissions to unsigned jars. Application requested security permissions, but jars are not signed.
```

Finding out the algo is possible with `jarsigner -v -verify foo.jar`. Then you have to edit `java.security` file for your runtime and remove the algo from the `jdk.jar.disabledAlgorithms` line.

## Running

You need to install java runtime and `icedtea-web`. Apparently, the latter can work with various runtimes, and various runtimes may coexist, so watch closely for using the exact java version you need. In my case I issued `sudo pacman -S java-runtime=8` and removed all other java versions *(though I tinkered before that quite a bit manually, so it's possible that something else might be needed)*. An IPMI I worked with did work with java 8.

`icedtead-web` provides `javaws`, and that it works can be checked by running `javaws -viewer`, it should launch a graphics interface.

`javaws` can be used to run jnlp files, but you probably would want browser to have them open automatically.
