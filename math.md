# General

A time span with stops *(e.g. intervals between events)* is isomorphic to "path" from graph theory. A path is a graph without branches; i.e. a graph with two nodes of degree one *(first and last)*, and the rest of degree two.

# Algebra

`Irreducible polynomial`: a one that can't be reduced into multiple polynomials of a lower degree within the same field. Also, if a polynomial ƒ(x) have a root, that is `∃a, ƒ(a) = 0` then `x-a` is a factor of ƒ(x), i.e. ƒ(x) can be reduced.

# Groups

A field consists of commutative addition and product operations, and the product operation distributes over an addition.

* `order` — is a number of elements.
* `galois field` aka `finite field` — a field with finite number of elements. E.g. calculations GF(256) = GF(2⁸) are often used to convert bytes to bytes.
* `hamming distance` (`d`): minimal number of steps required to change one word to another, where "step" is a minimal replacement of a character. E.g. a subset = {1000, 0001} of a set with 4-bit words, has d = 4. On the other hand, a subset {0000, 0001} has d = 1. Let `r` be a number of errors occured. Some facts:
  1. Finding errors requires `d >= r`
  2. Correcting errors requires `d > 2r`
* `generator element` an element whose exponents generate every element of the group.
* `cyclic group` a group with `generator element`

Finite fields typically made out of prime number of elements. Otherwise multiplication might have bad side-effects.

# Ereasure codes

* `erasure` an error in known position.
* `hamming codes`: are using "control codes" at every power of 2 position that controls parity of a group of the information. Not really effective.
* `codeword` is a data + redundancy.

## Reed-Solomon (RS)

Quoting: «It turns out that there is a finite field with p d elements for every prime number p and every positive integer d». Whatever that means, a paper says that commonly Reed-Solomon is based on GF(2^8) = GF(256). It says, the RS concept applies to any field.

`linear code` let a vector `M = [x_1, .., x_m]` be message vector, and a m×n matrix `C` with entries in the field; then `MC = [x_1,…,x_n]` is a `linear code` with `m` message symbols, and `n` codeword symbols. In an example[1] first `m` columns of `C` is an identity matrix `m×m`; and as results `MC` is `M|P`, i.e. the original `M`, and some redundancy info `P`.

Exponent in `GF(x^n)` signifies a number of elements side-by-side. E.g. `GF(2^8)` consist of 8 numbers one-by-one.

Addition & multiplication are not the usual ones. Multiplication works as `(a×b) mod c`, where `×` is a usual multiplication, `mod` is a division remainder, and `c` is an irreducible polynom of power one greater then number of bits.

Basically, works like following: a sequence of bits, e.g. `1111` being represented as a polynom `1 * x^3 + 1 * x^2 + 1 * x^1 + 1 * x^0`. Then we find an irreducible polynom of a degree `n` higher, which we use to encode information. Then we can lose n bits of information, and, with some magic, still restore the `1111`. The magic of decoding is a big research field, with a bunch of different algos, and, seemingly, none is easy to describe.

# Abstract

* `Embedding`: an injection from smaller object to a bigger. In particular there could be many embeddings. See also "word embeddings".
* `Hamming distance`: given two equally-sized words, it's a num of substution operations required to change one into another.
* `Levenshtein distance`: given two words, it's a num of {insertion,deletion,substution} operations required to change one into another.

1: Notes on Reed-Solomon codes.

# Combinatorics

* Number of permutations of n is `n!`
* Number of *k permutations of n* is `n! / (n-k)!`.

# Probability theory

* when one probability leads to another, which leads to third, they are all multiplied
* when one probability leads to two different that both are of interest, the two are summed up
