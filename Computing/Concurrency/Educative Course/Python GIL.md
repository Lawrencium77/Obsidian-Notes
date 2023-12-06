As well as basing these notes on the [Educative course](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/NEy8L9DMX5p), I've also taken some ideas from [this](https://towardsdatascience.com/python-gil-e63f18a08c65) Towards Data Science article.

Additionally, [this](https://blog.devgenius.io/python-do-you-really-understand-gil-8944fcd773cd) article has some really nice info, although I didn't actually use it to make these notes.

```toc
```

## Brief Overview of How the Python Interpreter Works

This understanding forms the basis of why we need the **Global Interpreter Lock (GIL)**. On a high level, the interpreter works as follows:

1. The entire Python file is compiled into Python byte code, resulting in `.pyc` files.
2. This byte code is then executed line-by-line by the **Python Virtual Machine (PVM)**. The PVM is not a separate component but part of the interpreter itself, responsible for executing the byte code.

This is a departure from pure interpretation, where each line of source code is interpreted and executed all at once. But Python is still considered an interpreter language because the compilation is to an intermediate byte code, not directly to machine code like in compiled languages.

## Limitations of Python Interpreter and the Need for GIL

The standard implementation of Python, CPython, **can run threads concurrently but not in parallel**. This is due to its memory management, specifically reference counting, not being thread-safe. The Global Interpreter Lock (GIL) is introduced to overcome this issue.

Specifically, it is the [reference counting](../../../Coding/Python/Garbage%20Collection.md) that is not thread-safe; incrementing a reference count is an opportunity for a **race condition** (another thread could read the `refcount` in between a read and write).

One possible solution could be to associate a lock with each object. However, this approach would lead to several locks being needed, potentially causing deadlocks.

Instead, a single lock is used to provide access to the Python interpreter. This lock is the [Global Interpreter Lock (GIL)](https://en.wikipedia.org/wiki/Global_interpreter_lock).

## How the GIL Works

The GIL ensures that Python **byte code** is executed by one thread at a time. The GIL effectively makes byte code execution [atomic](Atomic%20Operations.md). We could also say that each execution of a Python byte code instruction is a "**critical section**".

**It's important to note that this atomicity is applied to the level of Python byte code instructions, not to high-level Python code.** For instance, something like `x += 1` is not atomic in Python, as it involves multiple Python byte code instructions. So this operation could be interrupted mid-flow, meaning we can still get race conditions; just not within the execution of a single Python byte code instruction. 

Note that actions such as **decrementing reference counts** are part of the memory management performed by the CPython interpreter, not part of the Python language's byte code instructions. They happen automatically, **behind the scenes**. The GIL is relevant because it ensures that these operations are not interrupted by other Python threads. It's as if these reference counting is tied up within a single byte code statement. 

When a thread wants to execute Python byte code, it needs to acquire the GIL. When it has finished a byte code instruction, it may or may not release the GIL. To make this decision, CPython uses a mechanism called "**check intervals**", where the interpreter periodically decides whether the current thread should release the GIL and give other threads a chance to acquire it. This happens every 5ms by default (this variable be checked with `sys.getswitchinterval()`). This allows CPython to balance between the need for thread-switching (to ensure that all threads get a chance to run) and the overhead of acquiring and releasing the GIL.
Note that this is an example of [preemptive](Cooperative%20vs%20Preemptive%20Multitasking.md) scheduling.

> [!INFO]
> In Python 3.7, the GIL is a boolean variable that is guarded by a mutex. Its implementation lives in the [ceval.h](https://github.com/python/cpython/blob/3.7/Python/ceval_gil.h) file, which can be nice to peek at!

## When to Use Multithreading

Since Python can run threads concurrently but not in parallel, **CPU-bound** code will have no performance gain with Python multithreading. Instead, it is best used for **I/O-bound** tasks. 

For CPU-bound tasks, if you want to make better use of computational resources of multi-core machines, you'll need to use `multiprocessing`.