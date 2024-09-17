Haskell-esque non-lazy web development lang that transpiles to javascript. Seems to be possible to develop for mobile as well.

# Differences from Haskell

Partially gotten [from here](https://github.com/purescript/documentation/blob/master/language/Differences-from-Haskell.md)

* Strict evaluation
* inability to stuff declaration and assignment into the same line, i.e. this: `i :: Int = 1` is legal in Haskell but not in PS.
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
* "Records" are not `data` and work somewhat differently. Records enclosed into braces and then their fields are referred to via a dot. But there's more to it. Given this example:
  ```
  foo = {a : 1, "b" : 2, "A" : 3, "A B" : 4}
  ```
  First two fields are referred to by `foo.a` and `foo.b` and are basically the same. But the other two can't be referred directly as such and instead this syntax is used `foo."A"`, `foo."A B"`.

# Misc

* tools:
  * [current list](https://github.com/purescript/documentation/blob/master/ecosystem/Editor-and-tool-support.md#editor-support), may be useful because some tools are deprecated by others.
  * `purs` the compiler
  * `spago` a build tool for PS, there are: stable non-developed *(Haskell-based)* and unstable actively developed *(PS-based)* versions.
  * `pulp` an older build tool for PS, was used together with `bower` before `spago`.
* "array comprehension": `import Data.Array` and then use e.g. `1 .. 5`.
* async API is provided by `Effect.Aff` and starts with `launchAff` which converts `Aff a → Effect a`.

# HTTPurple misc

A backend lib for processing http methods.

* URL path/queries parsing is called "routing"
  * routes are declared as a record passed to `mkRoute` function. The record content is basically constructing the URL. Example from the docs:
  ```
  route :: RouteDuplex' Route
  route = mkRoute
    { "Home": noArgs -- the root route /
    , "Products": "categories" / string segment / "products" / string segment
    , "Search": "search" ? { q: string, sorting: optional <<< string }
    }
  ```

# Halogen misc

A library for UI in html + js.

* HTML tags are created by calling a function that creates a tag and passing it two arrays: 1. properties for the tag 2. children. Example, if we want this HTML:
  ```html
  <div id="root">
    <input placeholder="Name" />
    <button class="btn-primary" type="submit">
      Submit
    </button>
  </div>
  ```

  we code it like this:

  ```haskell
  import Halogen.HTML as HH
  import Halogen.HTML.Properties as HP

  html =
    HH.div
      [ HP.id "root" ]
      [ HH.input
          [ HP.placeholder "Name" ]
      , HH.button
          [ HP.classes [ HH.ClassName "btn-primary" ]
          , HP.type_ HP.ButtonSubmit
          ]
          [ HH.text "Submit" ]
      ]
  ```
* underscores: when HTML and PS keywords clash, Halogen adds an underscore in the name, e.g. `type_`. But then Halogen has also shortcut-functions ending with underscore for when you pass no properties, so instead of `div [] …` you can write `div_ …`
