Haskell-esque non-lazy web development lang that transpiles to javascript. Seems to be possible to develop for mobile as well.

# Differences from Haskell

Partially gotten [from here](https://github.com/purescript/documentation/blob/master/language/Differences-from-Haskell.md)

* Strict evaluation
* explicit forall, i.e. `foo :: a -> a` type annotation will result in error, it has to be `foo :: forall a . a -> a`. It supports `∀` though.
* `String` and `Char` are UTF16 for some reason that goes back to JS.
* `[a]` syntax not supported, it has to be `Array a` or similar.
* in `data Foo = Foo {a :: Int}` the `a` is not globally visible and instead you refer to it via a dot as a property.
* function composition: `<<<` instead of `.`. That's because dot is the property accessor.
* deriving show, [here are details why/how this code works](https://github.com/purescript/documentation/blob/master/guides/Type-Class-Deriving.md#deriving-from-generic):
  ```haskell
  import Data.Generic.Rep (class Generic)
  import Data.Show.Generic (genericShow)

  data Foo = Foo{a :: Int}

  derive instance genericMyADT :: Generic Foo _
  instance showMyADT :: Show Foo where
    show = genericShow
  ```
* `return` replaced with `pure`

# Misc

* tools:
  * [currently list of tools](https://github.com/purescript/documentation/blob/master/ecosystem/Editor-and-tool-support.md#editor-support), may be useful because some tools are deprecated by others.
  * `purs` the compiler
  * `spago` a build tool for PS, there are: stable non-developed *(Haskell-based)* and unstable actively developed *(PS-based)* versions.
  * `pulp` an older build tool for PS, was used together with `bower` before `spago`.
* "array comprehension": `import Data.Array` and then use e.g. `1 .. 5`.
