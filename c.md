# Misc

* A regexp for a pointer arg for batch-replacement purposes: `["/_a-zA-Z>-]+`
* `poll` vs `epoll`: return from `poll` requires to do `O(n)` scan to find the FDs that got triggered, whereas `epoll` gives them right away at `O(1)`.

# Time measuring macros to quickly insert in a code

```
#define TIME_MEASURE_PROLOG(postfix) \
    struct timespec start##postfix;                 \
    clock_gettime(CLOCK_REALTIME, &start##postfix);

#define TIME_MEASURE_EPILOG(postfix)                                    \
    {   struct timespec end##postfix;                                   \
        clock_gettime(CLOCK_REALTIME, &end##postfix);                   \
        fprintf(stderr, "@@@ #" # postfix " code took %f sec\n", (end##postfix.tv_sec - start##postfix.tv_sec) + \
                (end##postfix.tv_nsec - start##postfix.tv_nsec) / 1e9);   }

// usage:
//     TIME_MEASURE_PROLOG(1);
//     some_stuff();
//     TIME_MEASURE_EPILOG(1);
```
