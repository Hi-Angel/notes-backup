# Misc

## Building

Needs to be compiled in separate dir, i.e. `mkdir build`, `cd build`, `../configure …`, `make`. The configure flags I used:

    ../configure --prefix= --build=x86_64-linux-gnu --disable-multilib --with-system-zlib --without-included-gettext --enable-threads=posix --enable-libstdcxx-debug --enable-libstdcxx-time=yes --disable-werror --disable-nls --enable-languages=c,c++,lto

For how to change CFLAGS [see this](https://gcc.gnu.org/wiki/DebuggingGCC). It's interesting to notes that if you have a built GCC, you can rebuild just one directory with the new flags *(e.g. `gcc/` dir)*.

### Testing built GCC without installing

If you're going to debug a C/C++ code, then from the build dir you can run `gcc/xgcc -B ./gcc …`, where the option `-B` shows where most of gcc binaries are. This way you may get "missing includes" when trying to compile files, however you can work it around by compiling preprocessed files instead.

However, if you need to test it in full, as in, all includes/libraries, there's no way other than to install. Though you can install into the build directory. Just do `mkdir install`, then pass to configure an option `--prefix=/path/to/build-dir/install`, and after having it built run `make install`.

# Debugging GCC

While in code, you always can make GCC to pretty-print current GIMPLE and whatnot. Take a look at `gcc/gdbinit.in` file for available debugging helpers. *(note, currently they're broken, [hopefully the patch-set fixing them will be accepted soon](https://gcc.gnu.org/ml/gcc-patches/2019-11/msg00884.html))*

# Optimizations internals

There's a bunch of options for printing successful optimizations, however the option that seems to cover the most *(as in, the rest of them can lack some the info you wanted)* is `-fdump-tree-all-*=*`. The `-fdump-tree-all-optimized=1` that I used prints functions many times as they go through various passes.

Tree-dump has may have mentions of various optimization passes. Available ones are documented here: https://gcc.gnu.org/onlinedocs/gcc-4.2.0/gccint/Tree_002dSSA-passes.html

However, actual supported passes you can get with `-fdump-passes`. Then, suppose you want a pass for `tailc`. Then use: `-fdump-tree-tailc-all=filename`.

A source file example that contains an optimization pass: `gcc/tree-tailcall.c`

## Backend IR

It starts with converting GIMPLE to a backend-specific IR, which is represented by a LISP-like syntax. The RTL for passes on it can be looked upon with `-fdump-rtl-all`.

Quoting:

>     (reg:m n)
> is an expression representing a register access, and
>
>     (plus:m x y)
> represents adding the expressions x and y.

…

> The m in the expressions denotes a machine mode that defines the size and representation of the data object or operation in the expression

Machine-codes example: `QI` — "Quarter Integer" mode, a single byte treated as integer. `HI` — "Half-Integer" a two-byte one.

### Optimizations

They're mostly same as in GIMPLE, but can do better job with the target arch knowledge. For example, dead-code elimination may see that the operations working on the upper part of n-bit variables.

## Glossary

* **GSI** apparently `gimple_stmt_iterator`

## Links

* GCC Architecture Internals https://en.wikibooks.org/wiki/GNU_C_Compiler_Internals/GNU_C_Compiler_Architecture

### GCC Developer blogs

* https://kristerw.blogspot.com/
* http://hubicka.blogspot.com/
* https://developers.redhat.com/blog/2019/10/11/a-upside-down-approach-to-gcc-optimizations/
* https://developers.redhat.com/blog/tag/gcc/
