# Monads of multiple args

Given that `(>>=)` type is `m a -> (a -> m b) -> m b`, where would additional type-parameters go?

The way it works is via carrying, where the last parameter is the "monadic return value", whereas the rest of them are hidden. It's best seen with this snippet deriving the bind:

```haskell
instance Monad (MyMonad a) where
  -- may also write (>>=) :: MyMonad a b -> (b -> MyMonad a c) -> MyMonad a c
  (>>=) :: (MyMonad a) b -> (b -> (MyMonad a) c) -> (MyMonad a) c
  (MyMonad a b) >>= f = f b
```

Parentheses are for demonstration. The `m` here is `MyMonad a` that lacks the last type parameter. So the type `m x` would become a `MyMonad a x`.

It works similarly for `pure`, etc.

Now, to the question "what happens to the hidden parameters": that's just to the monad. A monad may even drop them completely if so desires.
