# Misc

Vanilla Emacs defines minor-modes through `easy-mmode` package. `smerge-mode` is one example, its `smerge-next`, `smerge-prev` are defined though `easy-mmode` mode.

# Tasks

## Unrecognized entry in undo list undo-tree-canary

Some info I gathered:

* the `undo-tree.el` package logs actions in `buffer-undo-tree` var, and then somehow they get transferred to `buffer-undo-list`.
* the error is about `buffer-undo-list` being set to just `(nil undo-tree-canary)`. The only place, I think, that could result in that is the function `undo-list-transfer-to-tree ()`, which has a line `(setq buffer-undo-list '(nil undo-tree-canary))`.
* `undo-tree.el` has only 2 functions that deal with `undo-tree-canary` thing.

## Emacs resize works wrong in KWin

It's due to `program specified resize increment` in `WM_SIZE_HINTS`. Works in other WMs simply because they ignore it *(which is bad though)*.

Can be worked around by setting `frame-resize-pixelwise` to `t` before frame is created.

Being set with `size_hints.width_inc` and `size_hints.height_inc` in Emacs code. First occurrence is found in `xterm.c`, commit `Initial revision` in 1991 year, function `x_wm_set_size_hint`. First occurrence in GTK related file is at `gtkutil.c`, commit `GTK files gtkutil.c and .h` in 2003. Both commits lack any description, and no comments on the resize matter provided.

I sent a patch to fix it, but turned out someone indeed needs it https://debbugs.gnu.org/cgi/bugreport.cgi?bug=36250
