## Futures

In Python's [asyncio](../../Computing/Concurrency/Educative%20Course/Event%20Loops.md), a `Future` is an awaitable object that represents the eventual result of an asynchronous operation. 

In other words, it's a special object that acts as a placeholder for the result of a computation that may not have completed yet. 

Here's a brief example:

```python
async def set_after(fut, delay, value):
    await asyncio.sleep(delay)
    # Set value as a result of Future.
    fut.set_result(value)

async def main():
    loop = asyncio.get_event_loop()
    fut = loop.create_future()
    loop.create_task(
        set_after(fut, 1, '... world'))

    print('hello ...')
    print(await fut)

asyncio.run(main())
```

In this example, the `set_after` coroutine sets the result of the future `fut` after a delay of 1 second. The main coroutine prints `hello...`, waits for `fut` to have a result, and then prints that result.

## Tasks
In high-level asyncio code, you often use `Task`s rather than `Future`s. `Task` is a subclass of `Future`. A `Task` is essentially an object that represents a computation that will be completed eventually. It runs a coroutine in an event loop and allows you to keep track of when it's completed.

