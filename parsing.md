# Misc

* DFA vs NFA *(Deterministic Finite Automatons vs Nondeterministic Finite Automatons)*: ± the same. DFA allows a single state at a time, while NFA allows to have multiple states at the same time. But DFA may negate the difference through a state that is the combination of the NFA states. So in the end the only difference will be that NFA is less code compared to DFA, but then DFAs are easier to debug/understand.
* "Word boundary"-feature absence in a parser: can be emulated by greedily consuming aphanumeric chars + optional whitespace. This works because: in case word may be followed by no space but by braces and whatnot, the braces isn't an alphanum, so it works. In case a word should be followed by another one, the space isn't optional, but it doesn't matter because you greedily consumed the word anyway. Gotcha: consuming the exact length *(e.g. exact word)*, this breaks because that isn't greedy.

# Chomsky's grammar levels

* **Regular Grammar**, Level 3: they use regular expressions, consist of symbols of an alphabet (`a`, `b`), their concatenations (`aa`, `ab`, `bbb`), alternatives (`a|b`). Can be implemented e.g. as a Finite State Automata.
   Can't handle nested expressions (such as `(()())`). It's because automata to deal with it should have inifinite number of states, since expressions can be nested infinitely.
* **Context Free Grammar**, Level 2: they can have nested, recursive, self-similar branches in their syntax trees, so they can handle with nested structures well. Implemented as a top-down, recursive-descent parser which uses machine's procedure call stack to track the nesting level.
   Can't handle context-sensitive expressions, e.g. when inside `x+3` the `x` can be a variable name in one context, and a function name in another.
* **Context-Sensitive Grammar**, Level 1.
