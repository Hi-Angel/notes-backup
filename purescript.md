Haskell-esque non-lazy web development lang that transpiles to javascript. Seems to be possible to develop for mobile as well.

May be installed via npm: `npm -g install purescript spago`

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

# …vs TypeScript

* In TS `const` and `readonly` works arbitrary. It won't disallow you to assign into `const` object or an array.
  * separately worth noting that even that is implemented poorly. If you want to declare a `const` parameter in a function, you can't just declare it `const`, but you instead have to do that via generic `function f<const T extends string>(arg: T)`.
* In TS equality works arbitrary. Comparing objects of different classes or interfaces or `type`s ignores their types, and just looks up the fields. If they match, you'll get no type mismatch. Worth pointing out it also stands for function parameters, not just comparisons. This has huge implications, because not only simply class objects are unreliable, but also *(or rather "especially")* unions of different types. And you also can't declare a simple wrapper type such as `Degree` vs `Radian`, well not without some tricks with `unique symbol` field.
* TS has exceptions. This is a large separate topic, but exceptions are generally frowned upon. Rust doesn't even include them.
* TS has no syntax for do-notation
* TS has no currying. "Who cares?" you might say, but bear with me: I looked at some production TS code using React, and it seems TS React programmers frequently create chains of `lambda` calls just because the lack of currying. So the feature is actually needed.
* ADTs *(like `data`)*are complicated to implement. You basically have to write a union, where each field is an object type with an explicit tag.
* There are some problems with Higher Kinded Types. An example of HKT is `Array<T>`, where type `Array` takes a type-parameter `T`. Now, `Array` is a predeclared HKT, but creating a new one such as `Functor<T>` results in some problems, which I can't give details about because I didn't dig into it *(just read that in a post with possible workarounds)*.

# Misc

* tools:
  * [current list](https://github.com/purescript/documentation/blob/master/ecosystem/Editor-and-tool-support.md#editor-support), may be useful because some tools are deprecated by others.
  * `purs` the compiler
  * `spago` a build tool for PS, there are: stable non-developed *(Haskell-based)* and unstable actively developed *(PS-based)* versions.
  * `pulp` an older build tool for PS, was used together with `bower` before `spago`.
* "array comprehension": `import Data.Array` and then use e.g. `1 .. 5`.
* async API is provided by `Effect.Aff` and starts with `launchAff` which converts `Aff a → Effect a`.
* `logShow` *(from `Effect.Class.Console`)*: to print from `Aff` to console. At least with HTTPurple it shows up in terminal as expected.

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

# Halogen

*(note: don't use it, use React instead, see the comparison further below)*

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
* caching: inside component `handleAction`, if `modify_ \state -> …` is called, the `state` is the cache. It will later be the input to `render`.

## Parent-child messaging

First, bad news: if you had read Halogen praises about how it's good in type-safety and well designed, well, this is where that ends. The messaging part is a bunch of useless abstractions where you can easily forget something and stuff silently breaks. For example, you can forget to insert the useless `receive = Just <<< Foo`, and everything will compile just fine but child won't be receiving inputs the parent sends it. Usually, with such huge abstractions you'd expect support for monkey-typing because there's too much to bear in mind, but for some funny reason Halogen is exactly the framework where it doesn't work. You have to study all those useless abstractions and make sure you got them right, or expect hours in debugging.

Given two components *(things created by `mkComponent`)*, they can exchange kind of like signals with optional data. Here, the parent is the component that inserts the other one via `slot` function.

`slot` inserts a component similarly to how a `HH.someTag` would insert a tag. Args: given a call `slot id subId component input mapChildOutputToParent`:

* `id`: a unique slot name defined via type-level magic like `_button = Proxy :: Proxy "button"`. The text should match the name inside `type Slots = ( button :: ButtonSlot Unit )`, where this `Slots` type is being used in the parent's `render` and `handleAction` functions.

  Purpose: it's given to `H.tell` function to send some signal/data to the slot.
* `subid`: in case you'd like to render the component multiple times, you can distinguish them by `subid`. Pass `unit` if not interested.
* `component`: the child created by `mkComponent`
* `input`: data to be passed to child's `initialState`
* `mapChildOutputToParent`: a function that takes child output and produces a parent "action", the one that `handleAction` takes as parameter. Typically it's just a parent-action data constructor that wraps the output.

`slot_` is similar to `slot` but with the output omitted, for cases where the child doesn't produce anything a parent would be interested in.

### Input

Child must provide to `H.defaultEval` a field `receive = Just <<< Foo` where `Foo` is a data constructor `ChildInput -> ChildState`, and then the data constructor is handled as the parameter to `handleAction`. The `receive = …` is the key — you'd think `handleAction` should be enough, but apparently Halogen authors decided it should be confusing and error-prone, so they introduced this useless proxy-method that serves no purpose and you can easily forget to add it. Though, it can be omitted when input is only provided via `tell` *(so e.g. you initialize the slot with `unit`)*.

With that out of the way, there are two ways to give an input to a child:

1. Implicit *(bad)*: inside parent you call `H.modify_` to modify some state that inside `render` gets passed to the child. This is error-prone, because if compiler determines the state wasn't modified it won't trigger the child. This is a problem when you have no input for the child but just want to signal it. But it's also a problem for when there is an input but child does something besides. Imagine invoking a modal window for certain data. If a user dismissed the window, Halogen won't ever invoke it again till the data changes.
2. Explicit *(good)*: parent calls `tell id subId QueryConstructor` where first two parameters are mentioned before and `QueryConstructor` is a constructor that may or may not pass some value, but the last mandatory argument you leave empty and then it's returned as `pure (Just next)`. Idk what it's for.

   For this to work you need to declare a separate function `handleQuery`, similar to `handleAction`, but taking the `QueryConstructor` type instead. Apparently `handleAction` wasn't enough for the authors, so now you need to bear `handleQuery` in mind as well, because if you forget to declare it thinking about `handleAction`, stuff will just silently break. Example:

  ```haskell
  data ButtonQuery a = SetEnabled a
  mycomponent = …
    handleQuery :: forall a . ButtonQuery a -> H.HalogenM ButtonState ButtonAction () Unit m (Maybe a)
    handleQuery (SetEnabled next) = do
        …do something…
        pure (Just next)
  ```

# React

Libs are ultimately an FFI-shim to a JS library, implying that if you ever get stuck beyond the basics, you can often search for solutions in JS-field and interpolate to PS. There're also examples [here](https://github.com/JordanMartinez/purescript-cookbook/tree/master/recipes), see dirs with "react" infix.

There're two implementations: `react-basic-classic` and `react-basic-hooks`. The "classic" is a class-based implementation *(pun is noted)* that predates "hooks". Nowadays "hooks" are preferred.

* atomic nodes *(a button, label, etc)* are represented by `JSX` type.
* "component" is a single React-managed DOM-tree *(made of `JSX`es and handlers)*
  * `Component` is the type, which is an alias to `Effect (props -> JSX)`. The `props` is an arg to be passed when instancing the component with `renderRoot`.
  * Running `Component` is done by unwrapping from `Effect` and passing over to `renderRoot`.
* "root" is a location for the first component to attach with `renderRoot`. Created by `createRoot`. There may be many roots.

  Bear in mind, just nesting components doesn't require creating new roots.
* Nesting components example *(a label inside a div)*:
  ```haskell
  labelComponent :: Component Unit
  labelComponent = component "Label" \_ -> do
    pure $ R.label_ [ R.text "Hello, world!" ]

  divComponent :: Component Unit
  divComponent = do
    c :: (Unit -> JSX) <- labelComponent
    component "Div" \_ -> do
      pure $ R.div_ [c unit]
  ```

## …vs Halogen

* Much simpler. You can read around for comparison, but in short: Halogen requires you to build inconvenient and error-prone abstractions; code reuse with Halogen is complicated. Making a generic component that accepts parameters and returns something back is so inconvenient that unless you come up with some crafty wrappers, you'll find easier to write a component each time anew than factor out existing ones to something generic.
* React has special `CSS` type, whereas Halogen has just a string instead.
* Halogen doesn't allow to execute `Effect` before rendering the initial state. So you have to jump through the hoops by assigning useless "initial state" which gets immediately replaced by the actual state in `handleAction`&co. In React you just execute what you need in `Component` and then pass it over to the lambda that will be creating the component.
* React elements *(`JSX`es)* are `Monoid`, Halogen's aren't. This simplifies conditionally rendering elements: instead of doing a `[multiple, children] <> if a then [anotherElem] else []` you just write `[multiple, children, guard anotherElem]`, where `guard` is the Monoid's. Much shorter, huh?
* Halogen's "Ref"s require you to name them, whereas React's don't. So React refs can't collide, whereas in a big Halogen project you can come up with name that was already used.

# Parsing

* Alternating branches via `<|>` should only be done at the top of do-block, otherwise failure wouldn't propagate properly. This works the same in original Parsec. [See this for details](https://github.com/purescript-contrib/purescript-parsing/issues/235#issuecomment-2692904083).

# Testing

* QuickCheck: for property-based testing, randomly generates tests that check given function properties.
* Spec: a usual testing framework. Provides different runners, one is node-based `node-spec` package.

## Spec

Basic example:

```haskell
module Test.Main where

import Prelude

import Effect (Effect)
import Test.Spec (Spec, it)
import Test.Spec.Assertions (shouldEqual)
import Test.Spec.Reporter (consoleReporter)
import Test.Spec.Runner.Node (runSpecAndExitProcess)

main :: Effect Unit
main = runSpecAndExitProcess [consoleReporter] spec

spec :: Spec Unit
spec = do
  it "adds 1 and 1" $ (1 + 1) `shouldEqual` 2
  it "adds 2 and 2" $ (2 + 2) `shouldEqual` 4
```

The `it` inside `spec` function are the separate tests.

There's also some `spec-discovery` for automatically discovering tests, but for me it wasn't finding some `output/cache-db.json/index.js` after following the docs and I didn't dig into that.

# Passing a pre-defined Record with all fields being optional

It may be desirable *(e.g. for FFI purposes)* to define some `type Props = {x :: Int, y :: Int}` but then to be able to pass just `{x: 7}` to some function, i.e. so `y` is not defined in the parameter.

It's useful to look at naive attempt first. Param here doesn't provide "predefined" fields, instead being "any possible record":

```haskell
foo :: ∀ a. Record a -> Effect Unit
foo = const unit

main = foo {x: 7}
```

We can improve it with some type-magic. PureScript has class `Union lhs rhs theUnion` which allows to declare a union of `lhs` and `rhs`. It works on `Row`s, but `Record` accepts one type-parameter that is a `Row`.

So we declare `Props` as a Row *(parentheses instead of curly braces)*, and in function declaration say that `Props` is a union of any two possible Rows; and then as the parameter we use `Record lhs` *(may also be `Record rhs`, doesn't matter)*, which basically says that the parameter is a record with any field enlisted in `Props`, which is exactly what we want.

```haskell
type Props = (x :: Int, y :: Int)

foo :: ∀ lhs rhs. Union lhs rhs Props => Record lhs -> Effect Unit
foo _ = pure unit

main = foo {x: 7}
```

It's interesting to note here that `Props` is used implicitly. The function parameter is just "some" type with type-constraint related to `Props`.

# Low-level debugging

`spago bundle-app` produces a JS, which may be run with `node` or a browser *(by augmenting it with `index.html`)*, and you can add `console.log()`s in it. In the JS stuff is getting renamed/mangled, but there is some structure to you can follow:

* `mkFnX` functions are translated to a `function (…) {}`. E.g. `mkFn2 \state2@(ParseState _ _ consumed) err -> …` is translated to `function(v4, err) { … }`.
* `runFnX` function are translated to `return someVal(…)`. E.g. `runFn5 k1 (ParseState input pos false) more lift foo done` is translated to `return v(new ParseState(v2.value0, v2.value1, false), more, lift1, foo, done);`
