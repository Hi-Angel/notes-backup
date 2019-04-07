# recording

By default stacks a recorded with frame-pointers, but those typically optimized out.

If app is built with libunwind, use `--call-graph dwarf` instead. There are other options.

# misc

There's lots of different events; all of them can be enabled explicitly with `-e my-event`.

Pressing "a" in perf-report shows record per assembly instructions *(and maybe source code)*. The percentage near instructions means "percentage of probes taken here compared to total within the function"; bear that in mind to make sense of those values.

In making `how to read perf-report` so far the best is to look at example in the bottom of `man perf-report`. The bottom part is: by default `perf` is using `--children`, but you usually want to look at `Self` column. So, to sort by `Self` pass a `--no-children` option — in this case "Overhead" col. should be the `Self`.

# Show the actual number of events that accords to a percentage

Use `-F+period` option. E.g. `perf report --stdio --no-children -F+period`
