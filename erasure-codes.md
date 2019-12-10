# Reed-Solomon codes

Nice overview of how these work can be seen [here](https://habr.com/ru/company/yadro/blog/336286/) *(it's in russian)*.

I use common matrix notation, where `m × n` means matrix with `m` rows and `n` columns.

## Encoding

Basically, you choose a generator matrix `m × n`, where `n` rows are the identity matrix, and the rest `m - n` rows are the input coefficients. The coefficients should be chosen in such a way, that you can remove any row from the matrix, and the resulting matrix is still invertible.

`Cauchy Matrix` is one that has these properties.

Then you multiply the matrix and `n` bytes of data represented as a column matrix. Due to how matrix multiplication works, this gives you `m`-sized column matrix, where `n` bytes are the original ones *(because the generator matrix has an identity at first `n` rows)*, and `m - n` bytes are so called "redundancy".

## Decoding

Let's call the `m × 1` vector we've gotten after encoding, `X`.

Decoding basically comes down to a small algebraic magic. Suppose we lost `2` bytes of data from `X` *(assuming, it's enough to still be able to restore it)*. We just remove the elements that correspond to the 2 bytes from the column vector, thus shrinking the matrix `m × 1` to `(m - 2) × 1`. Let's call it `Y`.

Likewise, we get the `(m-2) × n` generator matrix `H`, by removing the 2 rows from the original generator matrix `G`. And likewise, we get `H⁻⁻¹` inversion to the `H`. For completeness, assume `O` was the original `n × 1` data.

With all this defined, encoding from prev. section looked like this:

    G * O = X

We also know that:

    H * O = Y

To decode lost content, we get:

    H⁻¹ * H * O = H⁻¹ * Y ⇒
    O = H⁻¹ * Y

I bet this sounds too abstract, go look at the original habr article, it has much nicer illustrations.

## Implementation with ISA-L library

### `ec_init_tables()`

The input coefficients it accepts is the part of generator matrix that starts right after the identity part.

* `k`, actually `n_cols`: num cols in the generator matrix.
* `rows`, actually `n_rows`: num rows at the "input coefficients" part of the generator matrix.
* `a` actually `input_coefficients`: a pointer to input coefficients *(usually, part of the generator matrix that is after identity)*.
* `gftbls`, actually… ?
