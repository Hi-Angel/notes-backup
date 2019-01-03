# misc

```
/// All Lua objects can be cast to this.
#[derive(Debug)]
pub struct Object<'lua, S: ObjectStateType> {
    obj: AnyUserData<'lua>,
    state: PhantomData<S>
}
```

-----

* `protocols` dir contains some of wayland protocols.
* `awesome/src/build.rs` builds wayland protocols.
* generated from `xdg-shell.xml` code is brought in scope with `use wayland_protocols::xdg_shell::{…`. Occasionally or not, but it matches the path to `xdg_shell.rs` that extracts the generated code into itself. You also need to modify `wayland_protocols/mod.rs` to include the new module.

# todo

## global

* mouse state solving:
  1. ✓ Create a minimal noop protocol.
  2. Include it into awesome, and make it query the object somewhere at initialization state. Abort if object not found (it won't be just yet)
  3. Include it into way-cooler, and impl. the noop object.
  4. Add a request, and impl. on both sides, check that it's working.
  5. Impl. in Way-Cooler either interface or protocol "Way-Cooler", which should be for connection of Awesome, and would accept only one.
  6. Impl. some basic mouse state sharing.
* security: how about that Way-Cooler is the only one allowed to fork Awesome and give it access to connection fd?
* unify `way-cooler` and `awesome` as a single crate. It doesn't make sense to have them separated; and unifying them would allow to share some code.

## cleanup

* Probably remove `Default` for screen, and check for alike pattern in other structs. It doesn't make sense for screen to be in a "default state".
