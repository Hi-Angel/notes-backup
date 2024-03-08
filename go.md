# Dependencies and modules

Deps are managed for code that is considered a module, and apparently there's a new trend where you have to create module of anything you want deps to be managed for, even if it's a simple "hello world" with a dependency.

It's trivial though. Just do:

```
$ go mod init test     # this creates a `go.mod` file
[…]
$ go mod tidy          # this downloads all necessary deps
[…]
```

# Misc

* building: `go build -o ./a test.go`. Options should go before the source file.
* quasi-tuples: functions may return multiple values as `return 1, 2` which is declared as `(int, int)`. However Go does not consider them as tuples.
* when adding a `import ("some_link")`, pay attention that `some_link` may require more components, like `some_link/some_module`
* `:=` creates a new variable, but a `=` mutates an existing one
* in Go you often have an object of `interface`, to cast such object `iface` to a struct `st` use syntax `iface.(st)`. Go calls it a "type assertion".
* `interface`s are implemented implicitly. You just declare a struct that implements all methods that an interface has — and you're done.

# Language pros & cons

## Pros

* built-in LTO. Modules are downloaded as source code, so optimizer sees all code up to a point that goes out into non-Go library.

## Cons

* constants are basically non-existent. You can only ever declare `const` expressions that are evaluated at compile time. To make things worse, Go doesn't seem to support something like `constexpr`, so you can't make a function that should be evaluated at compile time.
* ternary do not exist. Which together with previous point forces programmers to write mutable code *(unless you're up to making a separate function for a one-liner that branches on `if`-condition)*.
* there's a nice pattern of functions returning a tuple *(though Go doesn't use this term)* of error and a "good value". However, it is impossible to declare a type of the components inside the tuple inside the assignment. You can do that on a separate line, which is of course verbose. So people simply don't do that, and as result reading a Go code is a guessing game, because you have no idea what types are involved. If you read it locally, you can fetch all modules and use "go to definition" to see the types *(which is long for a new project but at least works)*. But if you're reading it from a web-browser you are out of luck.
* `len` returns `int`. Sic. Where did they see length taking negative value? 🤷‍♂️
* instead of managing scopes like in Rust or C++ Go has a garbage collector
* using the absolute value function looks like this: `int(math.Abs(float64(-7)))`. Amazing.
* you can't allocate an `n`-sized array where all elements equal to some `X`. Instead you first allocate it with `make()`, then you initialize every element in a loop.
* `warning: ignoring go.mod in system temp root /tmp`: this is a warning you'll receive if you create a Go project in `/tmp`. That's one of the most frequent things to do: creating a project in `/tmp` to for bugreport-purposes and Go doesn't allow it. Amazing.
