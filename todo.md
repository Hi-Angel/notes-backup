# self-development

* Find out examples of total order (and other similar orders) to have useful isomorphisms in mind.

# software development

## zsh

* pasting long text makes it hang, probably because of completion. Make it use a timer to check whether text being pasted or typed.

## emacs

* tail-recursion optimization
* May be color-identifiers mode don't need a timeout after all? After all, syntax highlight works immediately.
*  *DEPRECATION: Duplicated values in array option "cpp_link_args" is deprecated. This will become a hard error in the future.* — wtf? Check if it gonna break CFLAGS and CXXFLAGS
* Lexical scope cause useless warnings in byte-compilation. Report that after making sure everything indeed works.
* Remove `global-highlight-symbol-mode`, and instead highlight whatever is selected *(without regexp)*.
* Make deletion/completion of keywords in one tap
* found, how to make emacs consume bunch of memory and hang. In xserver project, open `glamor/glamor_render.c` file, and try to use semantic to go to definition of `PictOpSaturate`. Emacs will quickly consume lots of memory, and then if you press C-g to stop it, it gonna hang.
* veeery slow, on vala files, profiler:
    ```
    - command-execute                                               11960  97%
     - call-interactively                                           11954  97%
      - funcall-interactively                                       11954  97%
       - self-insert-command                                        10501  86%
        - c-before-change                                            9237  75%
         - mapc                                                      9224  75%
          - #<compiled 0x155d516b7a01>                               9220  75%
           - c-before-change-check-unbalanced-strings                9166  75%
              c-pps-to-string-delim                                  8943  73%
            + c-syntactic-re-search-forward                           188   1%
            + c-literal-limits                                          8   0%
           + c-before-change-check-<>-operators                        40   0%
             c-parse-quotes-before-change                               7   0%
         + c-invalidate-sws-region-before                              10   0%
         + c-unfind-enclosing-token                                     3   0%
        + c-after-change                                             1139   9%
        + sp--post-self-insert-hook-handler                           119   0%
        + jit-lock-after-change                                         6   0%
       + backward-kill-word                                           996   8%
       + smex                                                         454   3%
       + evil-append                                                    3   0%
    + redisplay_internal (C function)                                 199   1%
    + timer-event-handler                                              31   0%
      internal-echo-keystrokes-prefix                                   4   0%
    + company-post-command                                              4   0%
    + evil-repeat-pre-hook                                              3   0%
    + evil-repeat-post-hook                                             3   0%
    + evil--jump-hook                                                   3   0%
    + highlight-symbol-mode-post-command                                1   0%
    + ...                                                               0   0%
    ```

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

## mesa

* the dispatch functions *(see also https://www.mesa3d.org/dispatch.html)* are defined and written in asm. I think, the code can possibly be optimized by a compiler if written i C, but defining a function in C may make a compiler to add `push`es at the beginning and what not. However it might be interesting to define a symbol in asm, but write the code in C. As a side effect this may be cross-platform.

## games to play with

* fortnite, it's "epic games", but doesn't have Linux version
* overwatch, MS's game, it's not free though.

# done

> * try to port all the `alloc.c` commits to remacs — possibly, it might fix the memory leak error. It turned out, Emacs being watched under valgrind by other peoples as well, and those unitialized values at alloc.c were guarded by macroses sort of `#IF VALGRIND_DEFINED`. But one have to explicitly configure in the valgrind support.

Tried porting, but there're complex conflicts. In addition I seem to have seen memory leak in vanilla Emacs under vague circumstances, which kind of defeats purpose of porting commits. All in all, I'm not motivated to solve the conflicts.

> * add transparency to Sway
>     * consider wiring up libanimation, adding transparency there instead

Opened issue https://github.com/swaywm/sway/issues/2787 It's been closed as it should work with a python script. WIP.

> * sway: rework transparency python script to keep track of existing windows, and not to issue commands too often. Then scratch up a PR to sway docs.

Done. Instead of docs it's added into a separate directory though.

> * In the `TRACE` macro use `unlikely` because typically that code isn't needed; also provide some benchmarks on branch hit'n'miss count.

Wine project rejected the patch with a note that it doesn't provide any measurable improvement.

[1]: https://stackoverflow.com/questions/2612447/pinpointing-conditional-jump-or-move-depends-on-uninitialized-values-valgrin
