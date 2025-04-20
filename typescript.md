# Project initialization

```bash
$ npm i typescript --save-dev # creates/adds to packages.json an entry of TS as a devel dep
$ npx tsc --init              # init the project and create tsconfig.json
```

Afterwards, plain `npx tsc` will compile files, assuming there are `.ts` files. Dirs searched for files by default are current + children, can be changed with a `"include": ["path1", "path2"]`.

# Misc

* `readonly` and `const` do same thing, but `readonly` is compile-time whereas `const` is both compile-time and run-time.
    * the two only work for primitive types and references. To make an object or array const use `Readonly<myobj>` or `Readonly<number[]>` or `ReadonlyArray<number>`.
    * `… as const` cast unexpectedly makes variable truly const. So e.g. `let myvar = [1,2,3] as const;` works same way as if it was `ReadonlyArray<number>`. But beware that if you add a conflicting type-hint in this case, it will override the cast. So to avoid bugs it's better to declare a variable with *either* type-hint or the cast, but not both.
* Question-marked field, e.g. `type Foo = { mbField?: string; }`: it's same type as `string | undefined`
* pattern-match: [use `ts-pattern` lib](https://github.com/gvergnaud/ts-pattern)
* "Mapped types" is a neat feature that allows to go over each element in array or object and apply some type-magic. Example: `type Readonly<T> = { readonly [P in keyof T]: T[P] }` makes every key of `T` object read-only.
* "Types conditioning": can declare type based on a type being passed: `type Condition<T, V> = T extends V ? number : string;`
* "Unit types": a value, can be used to implement `enum`s, e.g. `type A = "a" | "b" | "c";`
* `const foo = () => …` vs `function foo() …`: 1. the lambda has no its own lexical context, so won't have access to `this`. 2. the lambda can't be called before declared, but `function` can. 3. in lambda `return` may not be needed.
* Ambiguity of lambda body: if you return an object like `() => {something}` TS will be throwing errors because it thinks the braces are there simply to denote the body. Fixable with parentheses: `() => ({something})`.
* `class` vs `interface`: `interface`s work similarly to typeclasses in other langs *(so you use `class`es to implement `interface`s)*, except that additionally they may be used as a type itself.
  * `interface`s being used as a type may give problems if you wanted to extend pre-existing code. E.g. in React there's `interface CSSProperties` which is a monoid, except nobody implemented the function helpers. And then components has `style: CSSProperties`. And, well, while extending `CSSProperties` is possible, but it doesn't help given that you want the `style` to be of type that *implements* monoid, but interfaces can't implement anything.

      That said, it's more of a problem for library types *(because obviously if you want to change the type you have control over you just change it)*.
* `abstract class`: kinda like `interface`, but allows to have default implementations. Fields can be declared with `abstract` modifier, meaning they must be implemented in the child.
* "Object typing" syntax sucks. You can't declare a function `({foo: string, bar: number}) => {}` because this is not valid syntax. You know which one is? `({foo, bar} : {foo: string, bar: number}) => {}`. You can easily imagine how monstrous this becomes with a few more fields.
* Currying: `(foo) => (bar) => { … }` to make sense of it note that it is same as:`(foo) => ((bar) => { … })`, so you basically invoke it with `foo` and get a function expecting parameter `bar`.
* Hashmap: an object. Type is `Record<key, val>`.
    * `getOrDefault`: in TS/JS is implemented with `??` operator, e.g. `myMap[k]?? defaultValue`.

# React/TSX

* Given syntax `<Foo arg1=x>y</Foo>`, the `Foo` implementation is a function accepting an object `{arg1: SomeType, children: JSX.Element[]}`. The `y` is passed to special property `children` if you define it.
* Some props are deemed to be special. E.g. you can't make your own component that accepts `ref` even if it makes all rational sense. This isn't a problem e.g. in PureScript react-hooks library.
* "Self-closing tag": a tag like `<Foo/>` is different from `<Foo></Foo` in that the latter must have `children` field, even if empty.
* `useState`: returns a `[val, setVal]`, where `setVal` may be used to set the value. An important note here: `setVal()` may actually accept either a new value, or a function which would accept previous value and return new one.
