# Misc

* a signal like SIGTERM is only being sent to one thread, which supposed to communicate that to other threads
* `pthread_cancel` may cancel a thread, and then one can add cleanup handlers to be called upon cancellation request. Cancelling happens at certain library functions, like while waiting for a condition variable.
  It's a very annoying functional to use in multithreading apps, because if a mutex is locked when cancellation request came, the mutex is not going to get unlocked automatically, only if you created a handler for that specifically.
  A way better than using `pathread_cancel` might be just having a shared variable that threads periodically check.
