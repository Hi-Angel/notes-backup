Notes on various debugging tools and techniques.

# Profiling

## Perf

See `perf.md` file.

## Sysprof

A nice GUI utility with graphs and stuff, supporting both system-wide and not profiling, profiling CPU, memory allocations, IO accesses, power consumption, and other things. Can be used without GUI with `sysprof-cli`

As of latest nice tutorial is here https://blogs.gnome.org/chergert/2020/03/14/how-to-use-sysprof-to/ and https://blogs.gnome.org/chergert/2020/03/15/how-to-use-sysprof-to-part-ii/

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

Cool stuff you can do:

* run record till the point, where you know, some value is wrong. Then watchpoint it `wa -l some_value`, and run reverse-execution until the watchpoint gets hit: `rc`.
* sometimes watchpoints may cause more confusion, even combined with reverse-execution. Instead you can exploit another feature: every `rr replay` has same memory addresses. So you can set a bunch of breakpoints in suspicious places, and a `commands` to each one that e.g. only stops when variable has specific memory address.

# Sanitizer

## Algo for address sanitizer

In general it can be found here https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm In short:

* sanitizer uses some "shadow memory", where each byte shows state of 8-bytes block of app's memory.
* before or after every app's memory accesses inserted a guard that checks validity of the access.
* my experiments show, by default the "shadow memory" always resides at address `0x7fff8000`. This address is mentioned in asan docs, and is hardcoded in the compiler-generated code.

While debugging a complex sanitizer bug I wrote down below the actual implementation of stack-access checks by GCC 8.3.0 version.

Given this code:

    arr[i] = 0;

The assembly snip my GCC generated is:

    movq	-232(%rbp), %rdx	# arr, tmp127
    movq	-240(%rbp), %rax	# i, tmp128
    leaq	(%rdx,%rax), %rcx	#, _1
    # ../iscsi/asan_testcase.c:23:         arr[i] = 0;
    .loc 1 23 16 discriminator 3
    movq	%rcx, %rax	# _1, _19
    movq	%rax, %rdx	# _19, _20
    shrq	$3, %rdx	#, _20
    addq	$2147450880, %rdx	#, _21
    movzbl	(%rdx), %edx	# *_22, _23
    testb	%dl, %dl	# _23
    setne	%sil	#, _24
    movq	%rax, %rdi	# _19, _25
    andl	$7, %edi	#, _25
    cmpb	%dl, %dil	# _23, _26
    setge	%dl	#, _27
    andl	%esi, %edx	# _24, _28
    testb	%dl, %dl	# _28
    je	.L6	#,
    # ../iscsi/asan_testcase.c:23:         arr[i] = 0;
    .loc 1 23 16 is_stmt 0
    movq	%rax, %rdi	# _19,
    call	__asan_report_store1@PLT	#
    .L6:
    # ../iscsi/asan_testcase.c:23:         arr[i] = 0;
    .loc 1 23 16 discriminator 3
    movb	$0, (%rcx) 	#, *_1

pseudo-Haskell breakdown:

```haskell
let ptr_shadow_mem_base = Ptr 0x7fff8000
let ptr_arr_elem = address arr + i    -- arr and i appears somewhere higher
ptr_shadow_arr = (binary_shift_r 3 ptr_arr_elem) + ptr_shadow_mem_base
shadow_byte = get_byte ptr_shadow_arr
is_shadow_byte_set = shadow_byte != 0
smthng4 = shadow_byte >= (ptr_arr_elem & 7)
if is_shadow_byte_set != smthng4 then
    __asan_report_store1 ptr_arr_elem
else
    store_at 0 ptr_arr_elem
```

## Leak sanitizer

Works as follows: records every allocation/free, then before the end of the process scans entire process memory for pointers to recorded memory. Reports memory without pointers to it as leaked.

### Misc

* silently doesn't work under debugger. Including hooks like `__lsan_do_recoverable_leak_check` *(calling which may result in weird behavior such as exiting the process)*.
* leak detection may be run manually by calling `__lsan_do_recoverable_leak_check()` or `__lsan_do_leak_check()`, but note that the latter is confusing. It does not shut down the process, but instead it makes it so any further calls to leak checks *(including one that sanitizer will always do at the end of the process)* will no longer work. Better use `__lsan_do_recoverable_leak_check()`.
* "out-of-scope" leaks are mostly undetected. That is, if a pointer is still in memory but no longer reachable, it won't be reported. The only exception is if "out-of-scope" pointer is inside a function the code returned from and `detect_stack_use_after_return=1` set *(default since GCC-13)*.
* a possible reason for sporadic leaks may be that a thread was cancelled and didn't free memory nobody else has access to. Since threads have shared process memory, it is a valid leak for other threads.

# bpftrace

## Glossary

Probes, kprobes, tracepoint, etc. Refer to the table https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#terminology

An analog to Solaris `dtrace`. Syntax and options are different though, so scripts have to be ported between them.

## Misc

* `bpftrace` supports passing arguments to the script on command line. To print the first one, use `printf("%s", str($1));`. It's funny, that `$1` by default is an integer, i.e. you can print it with `%d` and you can not do it with `%s`. But applying a `str()` to it magically converts it from an integer to string. No idea.
* **Missing prints**: are usually caused by lack of `\n` at printf call. bpftrace has printfs newline-buffered, so it won't appear until either ^C or newline.
* To see args for a particular tracepoint use something like `sudo cat /sys/kernel/debug/tracing/events/syscalls/sys_enter_open/format`.

## Examples

* Trace all processes created in the system: `sudo bpftrace -e 'tracepoint:syscalls:sys_enter_exec*{ printf("pid: %d, comm: %s, args: ", pid, comm); join(args->argv); }'`
* Trace all threads created in the system: `sudo bpftrace -e 'kprobe:_do_fork{ printf("pid = %d, comm = %s\n", pid, comm); }'`
* Trace all `connect()` calls made in the system: `sudo bpftrace -e 'tracepoint:syscalls:sys_enter_connect{ printf("pid = %d, comm = %s\n", pid, comm); }'`

## References

1. Crush course: https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md

# strace

Example `strace -fe "openat,open" -T ls`. `-f` is for "follow forks", `-e` limits tracing to specific calls, `-T` shows time calls took in seconds resolution.
