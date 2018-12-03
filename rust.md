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

If a reference is mutable, only one is allowed within the scope. But multiple ones allowed by creating a new scope with curly braces!

## Slices

The syntax:

    let s = String::from("hello world");
    let hello = &s[..5];
    let world = &s[6..11];

It's a replacement to index, but it's better because if `s` been modified, we can forget to keep index in sync. However, here having a slice would disallow to mutate the string.

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

# wrapper types

https://doc.rust-lang.org/book/first-edition/choosing-your-guarantees.html

`Cell<T>` allows for mutability without mutable borrow. Mutation however makes a full copy of T every time.
`RefCell<T>` like `Cell<T>`, but provides read-write lock pattern at *runtime* with `borrow()` and `borrow_mut()` funcs, which makes sure that there's no other borrows active if a mutable one is taken *(otherwise it panics)*. Thread-unsafe.
`Rc<T>` a reference counted pointer, `T` is immutable.
`Rc<RefCell<T>>` a reference counted pointer to mutable data, which however can only have one mutable reference at a time.
