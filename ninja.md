# Building the ninja

`./configure.py --debug && ninja -C .`

# `build.ninja` syntax

There's a good manual, but other than that:

* indentation-level 0 allows keywords *(like `rule`, `build`, `pool`, etc.)*
* indentation-level 1 only allows variables assignment *(note that `rule` does not allow arbitrary ones, only predefined, as opposed to `build`)*
* variables:
  * content is a text
  * multiline text is supported by escaping the newline with `$` and continuing on the new one. Note that this means the newline character as well as indentation will be stripped. For example:
       ```
       multiline_variable = hello \n$
                            processing the \n character needs $
                            to be done by you (e.g. with `echo -e`)
       ```
* `rule` describes actions. It's basically an information that technically may not even be used at all.
* `build` is the actual actions carried using a `rule` and variables assignment
  * `build` syntax: `build output_file_path: rule_name inputs to rule`. The "inputs" are names of other `build` statements and so are explicit dependencies *(will be built before the current `bulid`)*. The rule will see inputs as `$in` and the output file/path as `$out` variables.
  * an implicit dep may be declared with `| my_other_build_rule`. This is useful when the dependency exists, however is not being an input, so you don't want it to be passed to `$in`.
  * to disable buffering of a `rule` or `build` statement use a `pool = console`
  * `phony` declares an alias, so e.g. `build easy: phony some/huge/name/foo.so` declares an alias `easy` to the longer target name. May also be used for grouping, like `build my_group: phony | target1 target2 target3`

# Process of picking files to rebuild

Ninja internally works with a tree, and considers whether nodes are "dirty" *(need a rebuild)* by various heuristics, including mtime. That happens at `DependencyScan::RecomputeOutputDirty()`

Build happens at `NinjaMain::RunBuild`, there you can see some usual prints, like "no work to do". Main building loop is at `Builder::Build()`

Next build action is taken from `Plan::` in `plan_.FindWork()` call. `FindWork` pops an edge from `this->ready_`.

Plan to build is at `want_` variable. It is an `Edge`, whose `inputs_` nodes are `.cpp` and `.hpp` files *(for a c++ project)*, and `outputs_` usually has one element: the object file, or a library, or something to build stuff into.

# Graphs

Work-in-progress section

Internally ninja uses a graph. Commentary at `graph.h` states it's a dependency graph. From what I've seen:

* Nodes: represent a target, which can be:
  * on top levels the target that is build by `ninja -C dir mytarget`
  * on lower levels these are files, such as `foo.cpp`.
  * leaf nodes are almost absent, and represent just libraries. Go figure…

# My current todos

## Constification of build graph

Ninja is using pointers in associative containers, which makes the body under the pointer to be non-const. I need to replace it with `const`, but problem ultimately that too many functions accept non-const.

To tackle it more easily it's better to start with constifying functions that work on Edge pointers taken from the key, and then finish it with constifying containers.
