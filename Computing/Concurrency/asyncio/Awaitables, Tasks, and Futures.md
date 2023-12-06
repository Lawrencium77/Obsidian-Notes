Having covered the [Basic Concepts](Basic%20Concepts.md), these notes describe the actual syntax used in Python.

```toc
```

## Writing Asynchronous Code
The most basic tool is the statement `async def`. This declares an asynchronous coroutine, in the same way that `def` defines a normal synchronous function.

An `async def` declaration looks deceptively similar to an ordinary `def` declaration. However, there are some key differences:

* The `def` keyword creates a callable object. When the object is called, the code block of the function is run. 
* The `async def` declaration also creates a callable object. However, when the object is called, the code block of the function is **not** run. E.g:

```python
async def example_coroutine(a, b, c):
  ...
```

means that `example_coroutine` is now a callable object. When you invoke it:

```python
r = example_coroutine_function(1, 2, 3)
```

this does not cause the code block to be run. Instead, an object of type `Coroutine` is created, and is assigned to `r`. To make the code block actually run you need to make use of one of the facilities provided by `asyncio` for running a coroutine. Common examples include `await` and `async.gather`. 

#### A Note on Terminology
We'll say a "coroutine object" for an object of class `Coroutine`.
We'll say a "coroutine function" for a callable that returns a coroutine object. 
When we need to refer to the code block inside a coroutine function, we'll refer to it as a "code block inside an `async def` statement which defines a coroutine function”.

## The `await` Keyword and Awaitables
The `await` keyword can only be used inside asynchronous code blocks. It is used as an expression which takes a single parameter and returns a value. For instance:

```python
   r = await a
```

A coroutine object is "awaitable". Recall that asynchronous code execution always happens in the context of a [Task](Basic%20Concepts#Events,%20Tasks,%20and%20Coroutines), which is an object maintained by the Event loop. 

The first time a `Coroutine` object is awaited, the code block inside its definition is executed in the current Task, with a new [stack frame](../../CSAPP/Chapter%203/Procedures.md) being added to the run-time stack. When the code block returns, the execution moves back to the `await` statement that called it. 

If a `Coroutine` object is awaited a second time, this raises an exception. 

Overall, we can think of awaiting a Coroutine object as being very similar to making a function call, with the notable difference that a Coroutine's object block can contain asynchronous code, and so can pause the current task during execution. 

There are three types of object that are awaitable:

* `Coroutine` objects. When awaited it will execute the code block of the coroutine in the current Task. The `await` statement will return the value returned by the code block.
* Any `asyncio.Future` class. When awaited, this causes the current Task to be paused until a specific condition occurs (see [next section](#Futures)). 
* Any object which implements the `__await__` method.

One of the most important points to understand is that the currently executing Task **cannot** be paused by any means other than awaiting a `Future`. And this can only happen inside asynchronous code. So any `await` statement *might* cause your current task to pause but is not guaranteed to. 

## Futures
Unlike a Coroutine object, when a Future is awaited it does not cause a block of code to be executed. Instead, a Future can be thought of as representing some process[^fn1] that is ongoing elsewhere and which may or may not be finished. 

When you await a Future, the following happens:

* If the process the future represents has finished and returned a value then the `await` statement immediately returns that value.
* If the process has finished and raised an exception then the `await` statement immediately raises the exception.
* If the process the future represents has not finished, the current Task is paused until it has. 

All `Future` objects `f` have the following methods:

* `f.done()` $\implies$ returns `True` if the process the Future represents has finished.
* `f.exception()` $\implies$ raises an `asyncio.InvalidStateError` if the process has not yet finished. If the process has finished, it returns the exception if one was raised.
* `f.result()` returns the value the process returned.

> [!IMPORTANT]
> The distinction between a Coroutine and a Future is important. A Coroutine's code will not be executed until it's awaited. A future represents something that is executing anyway, and simply allows your code to wait for it to finish, check if it has finished, and fetch the result if it has.


## Tasks
As described in the [previous article](Basic%20Concepts.md), each Event loop contains a number of Tasks, and every coroutine is executed inside a Task. So how to create a Task?

This is simple, and can be done entirely in synchronous code:

```python
async def example_coroutine_function():
    ...

t = asyncio.create_task(example_coroutine_function())
```

The `create_task()` method takes a Coroutine object as a parameter and returns a `Task` object, which inherits from `asyncio.Future`. The call creates the task inside the event loop, and starts Task execution at the beginning of the coroutine's code block. The returned future will be marked as `done()` only when the Task has finished execution.

Creating a task to wrap a coroutine is a synchronous call, so can be done inside synchronous or asynchronous code. If you do it inside asynchronous code then the Event loop is already running. When it gets the next opportunity it might make the new Task active.

However, when you make this call inside synchronous code, chances are that the Event loop is not yet running. Manually manipulating event loops is discouraged. Instead, if you do need to call a piece of asynchronous code in an other synchronous script, you should use:

```python
asyncio.run()
```

## Running async Programs
`asyncio.run(coro)` will create an Event loop, run `coro`, and return the result. It cannot be called when the Event loop is already running. This leads to a couple of obvious ways to run your async code.

This leads to an obvious way to run your `async` code, whereby we have everything is async coroutines and have a very simple entry function:

```python
import asyncio

async def get_data_from_io():
    ...

async def process_data(data):
    ...

async def main():
    while true:
        data = await get_data_from_io()
        await process_data(data)

asyncio.run(main())
```

Note that this doesn't make use of the ability of async code to work on multiple tasks concurrently. A more sensible example is given later on.

## How to Yield Control
There is not simple command for yielding control to the Event loop such that other Tasks can run. In most cases in an `asyncio` program, this is not something you want to do explicitly. Instead, we prefer to allow control to be yielded when you await a `Future` returned by some underlying library that handles some type of IO.

However, you do occasionally need to do so. There is a recognised idiom for doing so. The statement

```python
await asyncio.sleep(0)
```

will pause the current Task and allow others to be executed. The `asyncio.sleep` function returns a Future which is not marked as done until the specified number of seconds have passed. 

When using `asyncio.sleep` for $n > 0$ seconds, it's worth noting that your task will not always wake back up at $n$ seconds. Instead, it may wake up at any point *after* that time, since it can only awaken when there's no other Task being run on the Event loop.

## Making an Actual Program
The following is an example program which can instantiate multiple tasks and allows then to be swapped in and out:

```python
import asyncio

async def counter(name: str):
    for i in range(0, 100):
        print(f"{name}: {i}")
        await asyncio.sleep(0)

async def main():
    tasks = []
    for n in range(0, 4):
        tasks.append(asyncio.create_task(counter(f"task{n}")))

    while True:
        tasks = [t for t in tasks if not t.done()]
        if len(tasks) == 0:
            return

        await tasks[0]

asyncio.run(main())
```

This will run four Tasks which print the numbers from 1 to 99. After printing, each task will yield control to the Event loop. It produces output like:

```
task0: 0
task1: 0
task2: 0
task3: 0
task0: 1
task1: 1
task2: 1
task3: 1
...
```

The fact that the Tasks are scheduled in a round-robin fashion is not explicitly defined in the code. Instead, it's a consequence of how the Event loop manages Task scheduling. The Event loop could theoretically choose any Task to execute at any time but in practice, it ends up doing so in a round-robin fashion when each Task is identical.

## Summary
I think this summary diagram is kinda useful:

![](_attachments/Screenshot%202023-11-18%20at%2015.54.07.png)


[^fn1]: We mean this in a loose sense; it's not actually a different [process](../../CSAPP/Chapter%208/Processes.md)!