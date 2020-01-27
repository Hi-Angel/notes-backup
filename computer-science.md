# Formal language

* `alphabet` — a set of symbols that can be made into strings/words. Let's call it `A`.
* `letter` — an element of `alphabet`
* `words` (or synonym `strings`) — is a sequence of letters from `A`. All strings of length `n` is denoted `Aⁿ`, and all possible strings of any length is denoted $A^*$.
* `formal language` — is a subset of $A^*$ such that any given group of words represents a *well-formed* expression according to some rules.
* `non-logical symbols` — are symbols whose meaning depends on context. E.g. in sentence "let's call it `A`" above, the `A` is a one. It is an opposite to, for example, quantifiers like ∀ or ∃, whose meaning is always the same.
* `algebraic structure` — a collection of operations on A of finite arity, together with a finite set of identities, called axioms of the structure that these operations must satisfy.
* `signature` — describes non-logical symbols of a formal language. Formally, can be defined as a triple $δ = (S_func, S_rel, ar)$, where $S_func$ and $S_rel$ are disjoint sets. $S_func$ are function symbols, like ×, +, 0, 1. $S_rel$ are relation symbols or predicates, like ≤, ∈. $ar$ is a function $ar: S_func ∪ S_rel → ℕ$ that assigns *arity* to operations and relations.
* `formal grammar` — are rules to form strings that are valid according to the language's syntax. It can be used to generate strings, or to validate a string given. Example: `string grammar` can validate if a string is in the language or if it isn't.

# Algebraic Dynamic Programming

## Sources

1. [A video presentation](https://www.youtube.com/watch?v=anj_-Z2L4J0)
2. "A Discipline of Dynamic Programming over Sequence Data" — first paper on the matter.
3. "Implementing Algebraic Dynamic Programming in the Functional and the Imperative Programming Paradigm " — another version of the first paper by the same people. Parts of text are literally copy-pasted. It's shorter though and tries to provide more examples *(but IMO doesn't quite succeed)*

## Analyze (todo)

Let's try a simple `climibing staircase` problem. It sounds like:

> You are climbing a staircase that has `n` steps. Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

From 1: "the `signature` is a datatype at the root of every DP problem. It describes actions possible at any step." So: "go 1 step" and "go 2 steps" is the signature.

From video: `scoring algebra` *(evaluation algebra?)* is a function that for every type in `signature` returns some score. For the staircase problem we could return 1 for everything as we don't care *(we want enumeration)*
