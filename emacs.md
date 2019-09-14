# TAGS

Emacs loads that into a buffer, and parses whole buffer every time searching for a tag *(barring caches in various places through the stack)*. This is done by `find-tag-in-order` func. So if you wanna something like "extract all methods in file1", bad news for ya.

# Misc

Vanilla Emacs defines minor-modes through `easy-mmode` package. `smerge-mode` is one example, its `smerge-next`, `smerge-prev` are defined though `easy-mmode` mode.

# Contributing

This is hard. They're using some custom changelog format which you definitely not gonna want to do by hand. You may use hotkey `C-x 4 a` to produce changelog entried from currently uncommited changes, and then copy it to a commit.

# Tasks

## Unrecognized entry in undo list undo-tree-canary

Some info I gathered:

* the `undo-tree.el` package logs actions in `buffer-undo-tree` var, and then somehow they get transferred to `buffer-undo-list`.
* the error is about `buffer-undo-list` being set to just `(nil undo-tree-canary)`. The only place, I think, that could result in that is the function `undo-list-transfer-to-tree ()`, which has a line `(setq buffer-undo-list '(nil undo-tree-canary))`.
* `undo-tree.el` has only 2 functions that deal with `undo-tree-canary` thing.

# font-lock

To check font-lock-face set on the symbol/word under cursor use `describe-text-properties`.

font-lock is managed by `font-lock-keywords` list, which can store regexps as well as actual functions that do scan and stuff. Color-identifies is using that to scan a buffer.

## diff-mode: fixing up wrong line count when working with patchset

Problem: upon updating such header: `@@ -25,7 +25,6 @@` it sets the 2nd number *(i.e. 7 here)* wrong. It's one bigger than should be, because patchset ends with `--` thing, which Emacs considers as `removed`.

A patch sent https://debbugs.gnu.org/cgi/bugreport.cgi?bug=37395

# Done:

## Emacs resize works wrong in KWin

It's due to `program specified resize increment` in `WM_SIZE_HINTS`. Works in other WMs simply because they ignore it *(which is bad though)*.

Can be worked around by setting `frame-resize-pixelwise` to `t` before frame is created.

Being set with `size_hints.width_inc` and `size_hints.height_inc` in Emacs code. First occurrence is found in `xterm.c`, commit `Initial revision` in 1991 year, function `x_wm_set_size_hint`. First occurrence in GTK related file is at `gtkutil.c`, commit `GTK files gtkutil.c and .h` in 2003. Both commits lack any description, and no comments on the resize matter provided.

I sent a patch to fix it, but turned out someone indeed needs it https://debbugs.gnu.org/cgi/bugreport.cgi?bug=36250