Typical parser is based on using `pipe()`. Now, every param of `pipe()` barring the first one is a function that accepts a parser and returns one. This poses a problem: let's say you want to parse "many alphanums", so you do `S.many1(C.alphanum)` — how do you embed it into `pipe()`? The answer is wrapping it like `P.chain(_ => S.many1(…))`.

Another question is "how to return intermediate result from a pipe, i.e. as opposed to the last parsed value". A: you end pipe with `P.chain(result => … P.map(_ => result))`, where the `…` may be pipe continuation. So you basically end pipe prematurely by binding the "intermediate result", and you start a new pipe inside the lambda with the bind.

`apFirst` and `apSecond` helpers may reduce/simplify the code. `pipe(…, apFirst(myParser))` will run `myParser`, discard its successful result and return whatever was passed to it by pipe. A `pipe(…, apSecond(myParser))` does the reverse, i.e. discards previous successful result, runs `myParser` and returns its successful result.

# Reading the types

Figuring out what functions do you need from their types may be mind-boggling at first. The gist here is to note that combinators-design based on carrying the parameters. The core of the library if `pipe` from fp-ts/Effect libs. Your typical parsing snippet looks like `pipe(a, foo(b))`, note how `foo` is being invoked inside declaration. But `foo` invocation will actually return another function, which will accept `a`. IOW, it's equivalent to `foo(b)(a)`.

You might wonder: why not just call `foo(b)(a)` then? Well, real-world parsers will have many indentation levels:

```typescript
pipe(p1,
     p2(foo),
     chain(bar: Bar) => {
       do(something);
       return pipe(buzz,
                   p3(cuz)
                   …)
       })
```

Now, imagine you removed `pipe()` call and just moved `p1` to the bottom. The code will immediately become unreadable, because now you can't just go from top to bottom, but you have to jump back and forth between top and bottom and compare indentation levels.

With that out of the way, this is the reason that if you see a type declaration like this:

```typescript
export declare const apSecond: <I, B>(fb: Parser<I, B>) => <A>(fa: Parser<I, A>) => Parser<I, B>
```

You need to keep in mind that it is a carried function, which you'll declare in `pipe` as a combinator with arg `fb` and during runtime this function will accept `fa` and return result of running `fb` parser *(the `B` in the return type)*. So when you read the `pipe` call, the `fa` type will "feel" like the first argument, because it is indeed the first one to the combinator *(even though it is the second one in the function type)*.

# Misc

* `Parser` basically a function that accepts `Stream<I>` and returns `ParseResult<I, A>`. On the low-level it's a "callable interface".
* `pipe` from a different lib *(either fp-ts or effect)*: accepts a value, a then arbitrary number of functions, which are being chained together.
* `map`: maps one arg to another. Example:

   ```typescript
     P.chain(token => pipe(
       S.spaces,        // Trailing whitespace (0+ spaces)
       P.map(_ => token)  // Return the captured token
     ))
   ```
   This may look a bit confusing, because we parse spaces, and then return whatever came in before spaces, not after.
* `chain` is like `map` but allows to return failure from the parser. Accepts a lambda, where `chain(arg => …)` being used inside `pipe(…)`. May also help with wrapping a singular parser like `S.many1(C.alphanum)`, so you can embed it into `pipe(…)`, e.g.: `chain(_ => S.many1(C.alphanum))` *(otherwise the type wouldn't allow you to use it in pipe)*.
* `apFirst`, `apSecond`: to be used as body in `pipe(v, body…)`. They accept a parser, and `apFirst` discards its result and returns the previous one; whereas `apSecond` on the reverse, returns the parser result, discarding result that came prior.
* `seq` allows to combine result of two parsers. Note though that a `pipe()` for all but first param expects a `Parser  => Parser`, so `seq` call has to be wrapped with `map` or whatnot.
