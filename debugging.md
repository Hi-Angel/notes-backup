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

# Sanitizer

## Algo for address sanitizer

In general it can be found here https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm In short:

* sanitizer uses some "shadow memory", where each byte shows state of 8-bytes block of app's memory.
* before or after every app's memory accesses inserted a guard that checks validity of the access.
* my experiments show, by default the "shadow memory" always resides at address `0x7fff8000`. This address is mentioned in asan docs, and is hardcoded in the compiler-generated code.

While debugging a complex sanitizer bug I wrote down below the actual implementation of stack-access checks by GCC 8.3.0 version.

Given this code:

    foo[i] = 0;

The assembly snip my GCC generated is:

    movq	-232(%rbp), %rdx	# foo, tmp127
    movq	-240(%rbp), %rax	# i, tmp128
    leaq	(%rdx,%rax), %rcx	#, _1
    # ../iscsi/asan_testcase.c:23:         foo[i] = 0;
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
    # ../iscsi/asan_testcase.c:23:         foo[i] = 0;
    .loc 1 23 16 is_stmt 0
    movq	%rax, %rdi	# _19,
    call	__asan_report_store1@PLT	#
    .L6:
    # ../iscsi/asan_testcase.c:23:         foo[i] = 0;
    .loc 1 23 16 discriminator 3
    movb	$0, (%rcx) 	#, *_1

pseudo-Haskell breakdown:

```haskell
let ptr_shadow_mem_base = Ptr 0x7fff8000
let ptr_foo = address foo + i -- foo and i we got somewhere higher
ptr_shadow_foo = (binary_shift 3 ptr_foo) + ptr_shadow_mem_base
shadow_byte = byte_from ptr_shadow_foo
is_shadow_byte_set = shadow_byte != 0
smthng4 = shadow_byte >= (ptr_foo & 7)
if is_shadow_byte_set != smthng4 then
    __asan_report_store1 ptr_foo
else
    store_at 0 ptr_foo
```
