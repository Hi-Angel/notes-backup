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

1. `k`, actually `n_cols`: num cols in the generator matrix.
2. `rows`, actually `n_rows`: num rows at the "input coefficients" part of the generator matrix.
3. `a` actually `input_coefficients`: a pointer to input coefficients *(usually, part of the generator matrix that is after identity)*. Note: at the current version of ISA-L for some reason this parameter is mutable. But it does not get mutated.
4. `gftbls`, actually… ?

### `ec_encode_data()`

It may act in 2 modes: encoding and restoring. Sizes of buffers should satisfy constraints as noted in parameters description.

#### Encoding

Given payload, writes redundancy to the buffers provided.

1. `len`: length of each buffer *(all buffers must have the same length)*
2. `k`, actually `n_payload_bufs`: number of payload buffers, also must be equal to the number of columns in the generator matrix.
3. `rows`, actually `n_redundancy_bufs`: number of redundancy buffers, but it must be equal to the number of rows in the "input coefficients"-part of the generator matrix.
4. `gftbls`, actually… ?
5. `data`, actually `payload_bufs`: an array of pointers to buffers that contain payload.
6. `coding`, actually `redundancy_bufs`: an array of pointers to buffers, where to write redundancy.

#### Restoring

Given redundancy, writes "lost buffers" to the buffers provided.

1. `len`: length of each buffer *(all buffers must have the same length)*
2. `k`, actually `n_payload_bufs`: number of payload buffers, also must be equal to the number of columns in the generator matrix.
3. `rows`, actually `n_restore_to_bufs`: number of buffers to restore to.
4. `gftbls`, actually… ?
5. `data`, actually `left_bufs`: an array of pointers to buffers that contain left valid payload.
6. `coding`, actually `lost_bufs`: an array of pointers to buffers, where to write restored data.
