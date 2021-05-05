# Stacks

Per [this article](https://lwn.net/Articles/852112/), bpftrace lacks ability to map stack frames to functions if a sw was built without frame-pointers.

# Profiling

There're ebpf-based scripts for profiling called `bcc-tools` https://github.com/iovisor/bcc/ There's `-f` switch so their output can be passed to `perf` flamegraph.

## Profiling sleeping/locked/blocked times

There's a script `offcputime` that can give that. But even more interesting is `offwaketime` which combines stacks of the task that slept *(bottom half)* and the task that woke it up *(top half)*, separated with `--`. Note: order of execution of the top half is reversed compared to the other half, i.e. execution starts at the top and goes down.

Example *(profiling and creating a flamegraph)*:

```
offwaketime -kf 30 > out.txt
flamegraph --color=chain --countname=us < out.txt > out.svg
```

* there doesn't seem to be a way to run a profilee under one of these scripts, though they can be attached by pid.
* Amounts of time code took is often greater than the amount of time profiling took. I think it is because the time is a sum from various threads that may have executed the code. Like, if the same stack simultaneously appears in threads 1, 2, 3 and they're all blocked, I think in results is shown just one stack with the sum from all 3 threads. So if you measured 30 sec., and all that time they were blocked, you may get one stack with 90 seconds sleeping time.
  Not sure this is correct though, I did not check that.

# Deadlocks

`bcc` has a tool `deadlock` for detecting potential deadlocks. Haven't used it.

### References

* example in docs https://github.com/iovisor/bcc/blob/master/tools/offcputime_example.txt
* a blog post by Brendan Gregg on that matter http://www.brendangregg.com/blog/2016-02-01/linux-wakeup-offwake-profiling.html
