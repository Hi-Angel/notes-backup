# self-development

* Find out examples of total order (and other similar orders) to have useful isomorphisms in mind.

# software development

## emacs

* May be color-identifiers mode don't need a timeout after all? After all, syntax highlight works immediately.
*  *DEPRECATION: Duplicated values in array option "cpp_link_args" is deprecated. This will become a hard error in the future.* — wtf? Check if it gonna break CFLAGS and CXXFLAGS
* Lexical scope cause useless warnings in byte-compilation. Report that after making sure everything indeed works.
* Remove `global-highlight-symbol-mode`, and instead highlight whatever is selected *(without regexp)*.

## libinput

* libinput: improve jump detection to work with the record on the closed bug of mine.
  I measured, and the jumps don't get anywhere close to the thresholds libinput is using.

## wine

* Build with debug symbols, and run "perf" on h4mod.exe. There, probably, ddraw optimizations could be done.
* Use linked-list and other data structures from glibc, they're more likely to have a good performance than anything hand-crafted.

## sway

* Pointer lock doesn't work https://github.com/swaywm/sway/issues/1071#issuecomment-331705286
* Tray icons doesn't work *(possibly XEmbed protocol)*.
* Switching between windows stops working in non-english layout
* *(✓ implemented, on review)* Per-window keyboard layout would be nice https://github.com/swaywm/sway/issues/2361
* `parse_movement_direction` being run on every `focus` command. It should not be called, unless a command came from `swaymsg`.

# done

> * try to port all the `alloc.c` commits to remacs — possibly, it might fix the memory leak error. It turned out, Emacs being watched under valgrind by other peoples as well, and those unitialized values at alloc.c were guarded by macroses sort of `#IF VALGRIND_DEFINED`. But one have to explicitly configure in the valgrind support.

Tried porting, but there're complex conflicts. In addition I seem to have seen memory leak in vanilla Emacs under vague circumstances, which kind of defeats purpose of porting commits. All in all, I'm not motivated to solve the conflicts.

> * add transparency to Sway
>     * consider wiring up libanimation, adding transparency there instead

Opened issue https://github.com/swaywm/sway/issues/2787 It's been closed as it should work with a python script. WIP.

> * sway: rework transparency python script to keep track of existing windows, and not to issue commands too often. Then scratch up a PR to sway docs.

Done. Instead of docs it's added into a separate directory though.

[1]: https://stackoverflow.com/questions/2612447/pinpointing-conditional-jump-or-move-depends-on-uninitialized-values-valgrin
