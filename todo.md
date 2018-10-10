# self-development

* Find out examples of total order (and other similar orders) to have useful isomorphisms in mind.

# software development

* May be color-identifiers mode don't need a timeout after all? After all, syntax highlight works immediately.
*  *DEPRECATION: Duplicated values in array option "cpp_link_args" is deprecated. This will become a hard error in the future.* — wtf? Check if it gonna break CFLAGS and CXXFLAGS
* libinput: improve jump detection to work with the record on the closed bug of mine.

# done

> * try to port all the `alloc.c` commits to remacs — possibly, it might fix the memory leak error. It turned out, Emacs being watched under valgrind by other peoples as well, and those unitialized values at alloc.c were guarded by macroses sort of `#IF VALGRIND_DEFINED`. But one have to explicitly configure in the valgrind support.

Tried porting, but there're complex conflicts. In addition I seem to have seen memory leak in vanilla Emacs under vague circumstances, which kind of defeats purpose of porting commits. All in all, I'm not motivated to solve the conflicts.

> * add transparency to Sway
>     * consider wiring up libanimation, adding transparency there instead

Opened issue https://github.com/swaywm/sway/issues/2787 It's been closed as it should work with python script. I gotta find out how though.

[1]: https://stackoverflow.com/questions/2612447/pinpointing-conditional-jump-or-move-depends-on-uninitialized-values-valgrin
