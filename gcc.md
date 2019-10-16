# Misc

## Building

Needs to be compiled in separate dir, i.e. `mkdir build`, `cd build`, `../configure …`, `make`.

Using CFLAGS, CXXFLAGS have been resulting in stage-comparision failure. What have worked though, is using `BOOT_CFLAGS` instead.

# Optimizations internals

There's a bunch of options for printing successful optimizations, however the option that seems to cover the most *(as in, the rest of them can lack some the info you wanted)* is `-fdump-tree-all-*=*`. The `-fdump-tree-all-optimized=1` that I used prints functions many times as they go through various passes.

Tree-dump has may have mentions of various optimization passes. Available ones are documented here: https://gcc.gnu.org/onlinedocs/gcc-4.2.0/gccint/Tree_002dSSA-passes.html

However, actual supported passes you can get with `-fdump-passes`. Then, suppose you want a pass for `tailc`. Then use: `-fdump-tree-tailc-all=filename`.
