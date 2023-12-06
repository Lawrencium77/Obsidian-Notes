These notes contain a few important bits of information about multithreading.

* Threads operating within the context of a process. As such, they share the same VM space.
* However, each thread has its own [stack](Procedures#The%20Run-Time%20Stack).
* Each thread can also create its own segment, which doesn't share data with other threads. This is called **Thread-Local Storage (TLS)**.
* To allocate data in such memory in C++, we use the `thread_local` keyword, e.g:

```C++
thread_local int thread_specific_counter = 0;
```

* Each thread has its own separate **thread context**, which includes:
	* Thread ID
	* Stack
	* Stack pointer
	* Program counter
	* Register file (specifically, condition codes and general-purpose register values)
* It is impossible for one thread to access the register values of another thread. This is because the register values are swapped out each time we do a thread context switch.
* However, the memory model for separate thread stacks is not as clean. These stacks are contained in stack segment of VM, and are *usually* accessed independently by their respective threads. We say *usually* rather than *always*, since different thread stacks are not protected from other threads. So if a thread somehow manages to acquire a pointer to another thread's stack, then it can read/write any part of that stack.
