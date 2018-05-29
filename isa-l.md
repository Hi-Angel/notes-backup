# Misc

Bunch of low-level cryptographic-related (as opposed to just "cryptographic", i.e. because it's too low-level) functions, optimized for CPUs. Examples include API for hashing multiple buffers at a time through AVX and the likes.

Allegedly, acc. to their video intro, the library sets a CPU-optimized jump-table at runtime. Although in the little code I saw, CPU optimizations have been compile-time coded.

# erasure coding

Acc. to their video it haven't been widely used because of CPU-expensiveness; and the lib reduces the problem a lot.
