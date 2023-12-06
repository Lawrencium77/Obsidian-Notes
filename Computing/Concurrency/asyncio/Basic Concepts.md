These notes are based on [this blog](https://bbc.github.io/cloudfit-public-docs/asyncio/asyncio-part-1.html).
As a refresher, it could be worth reading my notes on [Coroutines and Subroutines](../../../Coding/Python/Coroutines%20and%20Subroutines.md).

## Coroutines in asyncio
In traditional subroutines, a function yields control only upon completion. Any later calls to the function are independent, and start from the beginning:

![](_attachments/Screenshot%202023-11-17%20at%2015.10.20.png)

Coroutines provide an alternative calling model. They use a different way for the method to move execution back to the caller: it "yields" control, but future calls to the coroutine will continue from where the execution left off. This way, control can bounce between the calling code and coroutine code:

![](_attachments/Screenshot%202023-11-17%20at%2015.11.57.png)

Python already has the capability for this execution model in the form of [generators](../../../Coding/Python/Iterators%20and%20Generators.md). But asyncio adds a new type of coroutine, which allows a natural way to write code where execution can move around between coroutines when one gets blocked.

## Events, Tasks, and Coroutines
In traditional multithreading, each thread has its own copy of the stack and stack pointer. These makes up two components of the thread context. 

Asyncio is single-process, single-threaded. Each thread possesses an object called an **event loop**. The event loop contains within it a list of objects called **Tasks**. Each task maintains a single stack and stack pointer. In this sense, tasks are analogous to threads.

![](_attachments/Screenshot%202023-11-17%20at%2015.16.39.png)

At one one time, the Event loop can only have one Task executing. All other tasks in the loop are paused. The currently executing task will continue to execute as if it were executing a typical synchronous Python program, up until it reaches a point where it would have to wait for something to happen before it can continue.

Then, instead of waiting, the Task **yields control**. The Task is paused by the Event Loop, and woken up again at a future point, once the thing it needs to wait for has happened.

The Event Loop can then select one of its other sleeping tasks to wake up and pick up execution. If none of them are able to be awaken, then it can wait.

## Python Multithreading vs Asyncio
In lots of ways, asyncio and Python multithreading are very similar:

* Threads and comparable to Tasks; both possess their own context, containing a stack and stack pointer.
* Only one Thread can operate at any one time (due to the [GIL](../Educative%20Course/Python%20GIL.md)). The same is true of Tasks.

However, the key difference between multithreading and asyncio is that:

> [!IMPORTANT]
> Multithreading uses pre-emptive scheduling.
> Asyncio uses cooperative scheduling.

As such, `asyncio` is essentially multi-threading where not the CPU but you, as a programmer (or really, your application), decide when a context switch happens."

Another difference is that Task objects are more lightweight than threads. This is because asyncio Tasks are not true OS threads; they're more akin to "green threads" or "user-level threads" managed entirely in Python. Task switching is therefore less costly than thread switching.








