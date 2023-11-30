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
