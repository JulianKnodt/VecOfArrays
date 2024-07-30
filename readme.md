# Vec Of Arrays

When creating algorithms for meshes, generally is _assumed_ to be in triangles, and thus only 3
indices need to be consecutively stored. When generalizing these algorithms designed just for
triangles, the first attempt at generalizing storing faces is a `Vec<Vec<T>>`, where each
nested `Vec` is a face, and T is a usize of the vertex index.
This performs poorly, because each face is its own allocation, and they
are not stored contiguously in memory. This means that iterating over sets of faces is slower
than if they were stored contiguously in one larger allocation.

Another attempt might be to use a `Vec<SmallVec<T>>`, where `SmallVec` is stored on the
stack. This performs better, but incurs `O(n)` runtime checks while iterating over faces.

An ordered vector of arrays incurs `O(1)` runtime checks, per set of indices of faces. Since
meshes will still contain mostly triangles, this is negligible and if there are only triangles
behaves nearly the same as `Vec<[T;3]>`.

## How it works:

The `VecOfArrays` struct is implemented as a flattened vector `Vec<Vec<T>>`, but each index
corresponds to an vector of elements. For example, the 3rd index actually corresponds to a
vector of `[T; 3]`, whereas the 2nd index corresponds to a vector of `[T; 2]`. If the size of
elements is known at compile time, then they can be inserted as fixed-sized arrays. If they are
known only at runtime, they must be inserted as slices.

#### Indexing:

To index into the vector of arrays, both the number of elements and the index within that arrays
of that size must be known. Indexing is done with `struct VecOfArraysIdx`, which contains an
index `idx` and the array size `n`. Other than `n`, this functions identically to standard
indexing.
