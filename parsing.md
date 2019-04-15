# Chomsky's grammar levels

* **Regular Grammar**, Level 3: they use regular expressions, consist of symbols of an alphabet (`a`, `b`), their concatenations (`aa`, `ab`, `bbb`), alternatives (`a|b`). Can be implemented e.g. as a Finite State Automata.
   Can't handle nested expressions (such as `(()())`). It's because automata to deal with it should have inifinite number of states, since expressions can be nested infinitely.
* **Context Free Grammar**, Level 2: they can have nested, recursive, self-similar branches in their syntax trees, so they can handle with nested structures well. Implemented as a top-down, recursive-descent parser which uses machine's procedure call stack to track the nesting level.
   Can't handle context-sensitive expressions, e.g. when inside `x+3` the `x` can be a variable name in one context, and a function name in another.
* **Context-Sensitive Grammar**, Level 1.
