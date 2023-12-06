I just made some quick notes to explain what an atomic operation is. I was motivated to do this when reading [these](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/xlm6QznGGNE) Educative notes.

In the context of [concurrency](Concurrency%20vs%20Parallelism.md), an atomic operation is one that:
* Appears to occur at a single instant between its invocation and response.
* Is guaranteed to completely fully or not at all.
* E.g. in the context of multithreading: completes in a single step relative to other threads. It cannot be interrupted by other threads.

This is important in concurrent programming where multiple processes or threads might access shared data simultaneously. 
If an operation is atomic, it can't be interrupted by another operation, and its result won't be dependent on the timing of other operations. This means they can be used safely in concurrent environments without needing additional synchronisation like locks or semaphores.