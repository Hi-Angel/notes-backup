# Misc

* Running is as simple as `spatch -sp_file my_rules.cocci -in_place foo.c`
* dry-run: it prints matches by default. So just run the above command without the `in_place` *(and without anything similar)*, and it should output potential changes without actually writing them anywhere.

A small example that replaces `bar1()` with `bar2` whenever it follows `foo()` call inside an if expression:

```diff
@ rule1 @
@@
if (<+...foo(...)...+>) {
    ...
-   ret = bar1();
+   ret = bar2;
    ...
}
```

# Troubleshooting

As of now coccinelle is not very good in reporting errors, almost everything you can imagine results in "parse error". Some mistakes I was stumbling upon:

* the `-` and `+` is not at the beginning of a line *(e.g. as a result of playing around with the file)*
* the `...` are not valid in a number of contexts.
* apparently, you must have at least one "modifying" rule, i.e. you can't just put a pattern without `-` or `+` to see matches.
* I've seen situations when `<+...` and `...+>` work, but not the `<...` or `...`. That is, despite that according to docs it should work. One such example is `if (...foo()...)` — it causes parse error, and `<...` works but does not match it.
* `spatch` have limited support for recursing over directories. This means, you can pass it a `dir1/ dir1/`, but only one of them will be processed. So if you want to pass more than one directory, you either gotta use the current dir, or pass it a list of files.

# References

* https://wiki.winehq.org/images/5/52/Coccinelle-wine-introduction.pdf nice collection of examples and brief explanations
