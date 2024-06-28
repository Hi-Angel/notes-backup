# Misc

* Often during parsing happens that both left and right sides of the regexp match. In this case left side is being preferred unless there's a lazyfying `?`, which makes the right one being preferred.
* Look-ahead/look-behind: *(perl syntax)* is a groups that starts with `?`, then another quantifier goes. Example: `(?=…)` and `(?<=…)`: positive look-ahead and look-behind, `(?!…)` and `(?<!)`: negative look-ahead and look-behind.
  * "Negative look-ahead example": matching a `this` in lines that do not contain `//` before `this`: `grep -P '^(?!.*//)^.*?\bthis\b' -- foo.c`. Here we first filter the line through the `^(?!.*//)` and then through the `^.*?\bthis\b`. Notice the `^` on the right side. It isn't really needed here, but I added it just to show that the left and the right sides are being matched separately on the whole line.
  * These groups are "non-capturing".
