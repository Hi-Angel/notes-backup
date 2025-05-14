# Misc

* "Natural transformation" is a family of morphisms, pictured by η. Morphisms are called "components". Given functors F and G and an object X, a component is a morphism in codomain F(X) → G(X). There's also commutativity requirement that basically saves the structure.

  Important distinction from a functor: a functor mapping *(whether between objects or their morphisms)* is an abstract concept that doesn't entail to be implemented by morphisms. E.g. a morphism between morphisms isn't a thing in a category of objects and morphisms. But η in "natural transformation" **is** a family of morphisms.

  A consequence is that `η: Id(X) → F(X)` is not the same as `F . Id = F`. That is, the functor composition is abstract, but the transformation are actual morphisms. Which in turn implies `η: Id(X) → F(X)` may not even exist despite `F . Id` existing.
* "Initial object": example is ø in the Set category *(see "empty function")*.
* Monad: an endofunctor F with η and μ natural transformations. "Endofunctor" basically says it can be infinitely composed, and then the 2 natural transformations create morphisms: η from "just objects" to "objects under functor" and μ maps from multiple applications to "one application less", up to just one.

# Exercises

`Seven Sketches in Compositionality` were suggested in the course of ETCH Zurich. They have solutions at the end of the book too, so one can self-check.
