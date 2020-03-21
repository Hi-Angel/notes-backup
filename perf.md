# Code Example

```
#include <stdio.h>
#include <stdlib.h>

int fib(int x) {
    if (x == 0) return 0;
    else if (x == 1) return 1;
    else return fib(x - 1) + fib(x - 2);
}

int main(int argc, char *argv[]) {
    // change counter to make it perform faster or slower
    for (size_t i = 0; i < 41; ++i) {
        printf("%d\n", fib(i));
    }
}
```

## Reading

### `fractal`

After running `perf report -g fractal -F+period`, percentages on a single align-level adds up to 100%.

```
-   99.98%    99.97%   12335987037  a        a                    [.] fib
   + 96.20% _start
   - 3.80% fib
      - fib
         - fib
         […]
           fib
         - fib
            - 66.88% fib
                 fib
                 fib
               - fib
                  - 50.54% fib
                       fib
                       fib
                       0xffffffff8d001a80
                    49.46% 0xffffffff8d001a80
              33.12% 0xffffffff8d001a80
```

# recording

By default stacks a recorded with frame-pointers, but those typically optimized out.

If `perf` is built with libunwind or libdw, use `--call-graph dwarf` instead. There are other options.

# misc

There's lots of different events that can be recorded with `-e my-event` option. For example `perf record -g -e branch-misses,cycles myapp`. To list them: `perf list`.

Pressing "a" in perf-report shows record per assembly instructions *(and maybe source code)*. The percentage near instructions means "percentage of probes taken here compared to total within the function"; bear that in mind to make sense of those values.

In making `how to read perf-report` so far the best is to look at example in the bottom of `man perf-report`. The bottom part is: by default `perf` is using `--children`, but you usually want to look at `Self` column. So, to sort by `Self` pass a `--no-children` option — in this case "Overhead" col. should be the `Self`.

## Show the actual number of events that accords to a percentage

Use `-F+period` option. E.g. `perf report --stdio --no-children -F+period`

## Flamegraphs

After a record is ready, `flamegraph` package can be used as `perf script | stackcollapse-perf.pl | flamegraph.pl > out.svg`.

Order: there's none. Don't count on right-to-left or otherwise columns location as meaning anything — they don't, it's random.

## Missing symbols

If you're reading `perf.data` from a PC different to one where report was created, you may want to give it a path to debug symbols. For debugging a Linux kernel module it helped me to do 2 things:

1. Give path to vmlinux file with `-k`
2. Give path to "kallsyms" with `--kallsyms=`. This file that resides in `/proc/kallsyms` file. Note that if you access a system remotely, this file may seem empty, so you need to copy it first.

## Measure an app execution time

`perf stat -r 10 myapp` would execute it 10 times, and print an overall statistics.

# links

Some discussion on formatting involving a perf developer https://lwn.net/Articles/379949/
