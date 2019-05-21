Notes on various debugging tools and techniques.

# Profiling

See `perf.md` file.

# gdb

See `gdb.md` file.

# `rr`, reverse execution and stuff

Well integrated with gdb, so you can use the usual gdb commands.

## Preparation

For faster execution either set `sudo sysctl -w kernel.perf_event_paranoid=1`, or make sure to have `CAP_SYS_ADMIN` on your user.

Dir to save record is `_RR_TRACE_DIR` *(from what I saw in the source, you can't modify the record name, only the directory)*. I set it to `_RR_TRACE_DIR=./`

## Running

* `rr record app app_args` to record
* `rr replay` to play the last record *(one pointed by `latest-trace` symlink)*

Cool stuff you can do: run record till the point, where you know, some value is wrong. Then watchpoint it `wa -l some_value`, and run reverse-execution until the watchpoint gets hit: `rc`.
