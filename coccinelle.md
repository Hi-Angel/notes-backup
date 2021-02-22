A terrible utility for C code refactoring. It was supposed to be a "smart C-code converter", into which you would throw requests like "inside every function foo(), swap fst and snd args", and then it will do as requested and handle complex stuff on the way, like indentation, nested constructions, etc. Well, that it does, and does very well. But there's a big problem, which cancels out completely these benefits of Coccinelle: 85% of SmPL code *(i.e. the code which explains what refactoring you want Cocci to do)* that you throw into it will not work, and you will never know why. Sometimes it will fail with vague errors that require you to reduce the SmPL-code to a working testcase, and then build up from there hoping you will catch the moment it starts complaining, so you'll know what caused it *(hopefully)*. Other times it gives no errors whatsoever, it simply doesn't do any replacement. Useless errors [are an architectural problem](https://github.com/coccinelle/coccinelle/issues/242#issuecomment-746646109), with which apparently nothing else can be done aside of rewriting Coccinelle from scratch.

So, most often you're better off writing a python script with `pyparsing` module. Though it probably won't handle unusual complicated code-constructions, but those rarely represent more than a few percents of the whole code-conversion, so you better off fix them by hand. Coccinelle is just not worth your time.

With that said, some of the stuff I found while trying to make it work is below.

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

# Examples

Replace any match of `!strlen()` with `strempty()` *(note the missing semicolons)*:

```
@ rule1 @
expression E;
@@
-   !strlen(E)
+   strempty(E)
```

# Troubleshooting

Coccinelle is terrible at reporting errors, so expect debugging to be mostly a guesswork. Some mistakes I was stumbling upon:

* the `-` and `+` is not at the beginning of a line *(e.g. as a result of playing around with the file)*
* the `...` are not valid in a number of contexts.
* apparently, you must have at least one "modifying" rule, i.e. you can't just put a pattern without `-` or `+` to see matches.
* I've seen situations when `<+...` and `...+>` work, but not the `<...` or `...`. That is, despite that according to docs it should work. One such example is `if (...foo()...)` — it causes parse error, and `<...` works but does not match it.
* `spatch` have limited support for recursing over directories. This means, you can pass it a `dir1/ dir1/`, but only one of them will be processed. So if you want to pass more than one directory, you either gotta use the current dir, or pass it a list of files.

# References

* https://wiki.winehq.org/images/5/52/Coccinelle-wine-introduction.pdf nice collection of examples and brief explanations
