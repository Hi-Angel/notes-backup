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
