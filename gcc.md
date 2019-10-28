# Misc

## Building

Needs to be compiled in separate dir, i.e. `mkdir build`, `cd build`, `../configure …`, `make`. The configure flags I used:

    export BOOT_CFLAGS="-O0" # we don't want to spend time optimizing debug version
    export BOOT_CXXFLAGS="-O0"
    ../configure --prefix= --build=x86_64-linux-gnu --disable-multilib --with-system-zlib --without-included-gettext --enable-threads=posix --enable-libstdcxx-debug --enable-libstdcxx-time=yes --disable-werror --disable-nls --enable-languages=c,c++,lto

Using CFLAGS, CXXFLAGS have been resulting in stage-comparision failure. What have worked though, is using `BOOT_CFLAGS` instead.

### Testing built GCC without installing

If you're going to debug a C/C++ code, then from the build dir you can run `gcc/xgcc -B ./gcc …`, where the option `-B` shows where most of gcc binaries are. This way you may get "missing includes" when trying to compile files, however you can work it around by compiling preprocessed files instead.

However, if you need to test it in full, as in, all includes/libraries, there's no way other than to install. Though you can install into the build directory. Just do `mkdir install`, then pass to configure an option `--prefix=/path/to/build-dir/install`, and after having it built run `make install`.

# Optimizations internals

There's a bunch of options for printing successful optimizations, however the option that seems to cover the most *(as in, the rest of them can lack some the info you wanted)* is `-fdump-tree-all-*=*`. The `-fdump-tree-all-optimized=1` that I used prints functions many times as they go through various passes.

Tree-dump has may have mentions of various optimization passes. Available ones are documented here: https://gcc.gnu.org/onlinedocs/gcc-4.2.0/gccint/Tree_002dSSA-passes.html

However, actual supported passes you can get with `-fdump-passes`. Then, suppose you want a pass for `tailc`. Then use: `-fdump-tree-tailc-all=filename`.

A source file example that contains an optimization pass: `gcc/tree-tailcall.c`

## Links

* GCC Architecture Internals https://en.wikibooks.org/wiki/GNU_C_Compiler_Internals/GNU_C_Compiler_Architecture
