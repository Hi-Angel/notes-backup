# Installing

* `rustup`: `rustup toolchain install beta`

# ownership

## Basics and moving

Take this code:

    let a = x; // where x is a constructor of some type discussed below
    let b = a;

Stack-only variables are copied, but for stack+heap vars (like `String`) only stack-part is copied, and the heap is shared.

In the latter `a` is "moved" to `b`, i.e. `a` is no longer valid. To force a copy in this case requires using `::clone()` method.

Stack-only variables are distinctive in having implemented trait `Copy**.

**In functions**, like, passing arguments, returning values *(which moves it up)* it works the same.

## "References" and borrowing

`&String` is a reference declaration *(e.g. in function decl)*, and `&s` would be passing value by reference *(it gotta be done explicitly)*. The `s` there would be borrowed.

References are always assigned lifetimes *(most of the times they're derived by compiler though)*. The syntax is as `fn bar<'a, 'b>(...)`, where `a` and `b` are lifetimes' names, which are then used in args such as `...(x: &'a i32)`

If a reference is mutable, only one is allowed within the scope. But multiple ones allowed by creating a new scope with curly braces!

## Slices

The syntax:

    let s = String::from("hello world");
    let hello = &s[..5];
    let world = &s[6..11];

It's a replacement to index, but it's better because if `s` been modified, we can forget to keep index in sync. However, here having a slice would disallow to mutate the string.

# type parameters/templates/generics

Function example:

    fn func<T: std::fmt::Debug>(a: T) {
        println!("{:?}", a);
    }

    fn main() {
        func(1);
        func::<u32>(1);
        func::<_>(1);
    }

# type-inference

Aside of C++-like, type can be inferred from return values. This is how `into()` works.

# packages

* `package` is a collection of crates.
* `crate` is a binary or library.
* `module`s are created by `mod foo{}` declaration, whereas included by just `mod foo;`, and are brought into scope by `use foo;`. `main.rs` and `lib` are creating a module `crate`.

# cargo tips

* `cargo` is a native build system for Rust. It handles dependencies, configuration, compilation… It builds a source code down the `src/` dir. Specifically, it compiles `main.rs` or `lib.rs`, and and everything referenced from there with `mod foo;`
* `cargo build --all` builds everything.

# code-generation

Cargo looks up by default for "build.rs" file, which gets compiled and executed for distinct preparations, such as generation of a code for handling Wayland protocols. The executed gets some environment variables set, such as `OUT_DIR`, etc.

# misc

Casting data to a struct, example:

    #[repr(C, packed)]
    struct my_struct {
        foo: u16,
        bar: u8,
    }

    fn main() {
        let v: Vec<u8> = vec![1, 2, 3];
        let s: MyStruct = unsafe { std::ptr::read(v.as_ptr() as *const _) };
        println!("here is the struct: {:?}", s);
    }

Rust's analog to `std::set<T>` is `HashSet<T>`. Apparently it's implemented as `HashMap<T,()>`.

# wrapper types

https://doc.rust-lang.org/book/first-edition/choosing-your-guarantees.html

`Cell<T>` allows for mutability without mutable borrow. Mutation however makes a full copy of T every time.
`RefCell<T>` like `Cell<T>`, but provides read-write lock pattern at *runtime* with `borrow()` and `borrow_mut()` funcs, which makes sure that there's no other borrows active if a mutable one is taken *(otherwise it panics)*. Thread-unsafe.
`Rc<T>` a reference counted pointer, `T` is immutable.
`Rc<RefCell<T>>` a reference counted pointer to mutable data, which however can only have one mutable reference at a time.

# flycheck-mode

Just call `flycheck-rust-setup` before enabling it.

# destructor

`Drop` interface is it.

# phantom data

`marker::PhantomData` is used in structs that hold and own pointers, to mark pointed-to type as owned. Otherwise, simply having a pointer in a struct does not attach any ownership, thus destructors of pointers not gonna get ran upon struct being destroyed.

# Debugging

## Abort on panic

You can either write in `Cargo.toml`:

```
[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

or run rustc with `-C panic=abort` option.
