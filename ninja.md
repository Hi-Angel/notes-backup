# Building the ninja

`./configure.py --debug && ninja -C .`

# Process of picking files to rebuild

Ninja internally works with a tree, and considers whether nodes are "dirty" *(need a rebuild)* by various heuristics, including mtime. That happens at `DependencyScan::RecomputeOutputDirty()`

Build happens at `NinjaMain::RunBuild`, there you can see some usual prints, like "no work to do". Main building loop is at `Builder::Build()`

Next build action is taken from `Plan::` in `plan_.FindWork()` call. `FindWork` pops an edge from `this->ready_`.

Plan to build is at `want_` variable.

# Graphs

Work-in-progress section

Internally ninja uses a graph. Commentary at `graph.h` states it's a dependency graph. From what I've seen:

* Nodes: represent a target, which can be:
  * on top levels the target that is build by `ninja -C dir mytarget`
  * on lower levels these are files, such as `foo.cpp`.
