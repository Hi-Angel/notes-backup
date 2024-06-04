A formal language for proving correctness of concurrent designs with many conditions and timings. TLA stands for Temporal Logic of Action.

# Misc

## Computer Science relevant concepts

* Properties of execution:
  * `safety property`: just an assertion.
  * `liveness property`: an assertion that something eventually happens. Opposed to `safety`, it does not claim that something is true "right now", only that it will be "true" at some point.
* `temporal logic`: a system of rules describing statements that may vary in time. So e.g. statement "I am cool" is constant in atemporal logic, but may sometimes be true or false in temporal.
* `\/` and `/\`: `or` and `and` accordingly.
* `=>`, `<=>`: `implies` and `equiualent`.

## General

* Many math operations are written like in latex, e.g. `\div`, `'\in'`, etc.
* The syntax allows unicode math symbols. So for "and" you can write both `/\` and `∧`.
* `PlusCal` is a DSL that's getting translated to TLA+. It may be more expressive for some tasks, and it supports embedding TLA+. The resulting TLA+ can be seen with `File → Translace PlusCal Algorithm`.
* `TLC` state checker, an app that analyzes if a `TLA+` scenario satisfies all props.
* `Fingerprint collision probability`: TLC stores explored states as hashes, so this is the collision probability. In practice it is always very low so can be ignored.

# TLA+

* `<=>`: similar to `=` but only works on booleans.
* **Types**: primitive: strings, bools, ints, model values; complex: sets, sequences, structures, functions.
  * **model values**: [often used to define NULL values](https://learntla.com/core/constants.html#model-values). It doesn't seem to be possible to define "model values" purely in the code. A model value `Foo` is created by declaring a constant `Foo` and then going to tab "Model overview" and in the widget "What is the model" editing the constant *(it will be shown in the list)* and assigning it a radiobox "model value".
  * sequences: basically an array, written `<<a, b, c>>`, access is 1-indexed, ex. `seq[i]`. Module `Sequences` has various operators such as `Append(seq, elem)`, `Head(seq)`, etc.
  * sets: an array of unique values, written `{1, 2, 3}`.
* Operators: basically functions, defined like `MinutesToSeconds(m) == m * 60`.
* Functions: a domain → codomain mapping, defined like `F == [x \in S |-> expr]`. For something that isn't a mapping see operators or macros. Functions are similar to Haskell functions, and they too allow `LET … IN` syntax, including declaring "subfunctions" in `LET`.
  * operators vs functions: notice how operators can be applied to a value. Functions can't, instead they define a codomain which you then refer to as e.g. `F[3]`.
* `LET`: similar to Haskell `let`, [allows to break function definitions](https://learntla.com/core/operators.html#let) to multiple lines.
* **IF-THEN-ELSE**: must be upper-case, for example `Abs(x) == IF x < 0 THEN -x ELSE x`
* there are maps and filters in set-builder notation.
* set filter: `{X ∈ my_set: condition}` e.g. `{x ∈ 1..4: x % 2 = 0 }`
* `CHOOSE`: pulls a value from set. The syntax is similar to set-filter, except the braces may be omitted: `CHOOSE x ∈ my_set: condition`
  * **NOTE**: `CHOOSE` is [the preferred way of getting a value that in a usual programming language would be calculated with ternaries and stuff](https://learntla.com/core/operators.html#choose). TLA+ presumably optimizes stuff out and it simplifies the model for the reader.

# PlusCal

* First line: `(* --algorithm FOO`, then you may put `variables` section, then `begin` and ends with `end algorithm; *)` line.
* `MyLabel:` a label encapsulating atomically happening code.
  * Any statement *(barring the initial variables declaration)* must belong to a label.
  * A variable can be updated only once within a single label. But as a way around there's `||` operator.
  * `goto Foo`: going to label `Foo`
* `EXTENDS a, b`: imports of `a`, `b` modules. Only one `EXTENDS` line is allowed.
* `:=` assignment, `=` is a comparison, `#` is same as `≠`.
* code within a single label is executed "atomically".
* `await (cond)` means waiting for `cond` to become true. But if `cond` contains multiple actions, they're not atomic and hence has to be in separate labels. Then it can be replaced with `while (TRUE) {cond}`.
* `macro`: a textual substitution similar to C macros. May not contain labels. Must be placed above `begin`.
* invariant: basically an assertion that something is true in every possible step. Defined in `define` block. It has TLA+ syntax but it can refer PlusCal variables. Example
    ```
    define
        MyInvariant == is_unique \in BOOLEAN
    ```

    Invariants are most useful with quantifiers *(like ∀, ∃, etc.)*

    **Important**: if an invariant ignored/doesn't work, that means you have to go to `Model Overview` tab, find `Invariants` widget and "Add" the invariant name there. Doesn't seem to be any way around that problem currently.
* Nondeterminism: allows testing multiple inputs each of which can potentially happen:
  * `with`: inside block below the `roll` can be any of the 6 values, so the checker will try all 6:
        ```
        with roll \in 1..6 do
          sum := sum + roll;
        end with;
        ```
  * `either`: says that control flow can take any enlisted state. In the snippet below TLC will create/check 3 branches:
        ```
        either
          approve_pull_request();
        or
          request_changes();
        or
          reject_request();
        end either;
        ```
* processes: anything concurrent *(threads, programs, computers, or even people for all you care)*. Syntax:
    ```
    process foo = 1 \* may also be a set like `foo \in {1, 2, 3}`
    variables       \* optional, may be used for process-local vars
    begin
      label:
        …algo…;
    end process;
    ```

    The `1` in the syntax defines process mapping in `pc`. `pc` maps process values to their labels, so e.g. retrieving foo's label can be done with `pc[1]`. It may be useful for writing invariants.

    * running forever: often processes don't run for a moment but are kept "forever". That can be implemented with either `while TRUE` or just `goto begin;`
* procedures: like macros, but may contain labels. Don't return values. Ex.:
    ```
    procedure Name(arg1, ...)
    variables var1 = ...
    begin
      label:
        \* stuff
        return;
    end procedure;

    /* and then inside a process:
    …
    call my_procedure(a, b);
    ```
* struct type *(and hashmap)*: declaration `my_struct == [ field1 |-> 1, field2 |-> {}]`, access: `my_struct.field1`. It is actually a hashmap and another way to access it is `my_struct["field1"]`

## Starting a PlusCal model

1. `File → Open Spec → Add New Spec` and enter a path ending with a file with `.tla` extension.
2. Type the algo in between the `---- …` and `==== …` lines.
3. If you use PlusCal, gen the TLA+ by pressing <kbd>Ctrl</kbd>+<kbd>t</kbd>
4. `TLC Model Checker → New Model`
5. In tab `model overview` edit `What's the behavior spec` to chose `Temporal formula` from the drop-down list, and then put `Spec` to the textbox right below.
6. Run `TLC Model Checker → Run model`

For simple evaluation purposes you can write an expression `Eval == <whatever code goes>`, and then in the model the `Evaluate Constant Expression` window type lone word `Eval` and press F11.
