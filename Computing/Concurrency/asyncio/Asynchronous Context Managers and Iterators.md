These notes are based on the [third part](https://bbc.github.io/cloudfit-public-docs/asyncio/asyncio-part-3.html) of the BBC blog.

## Asynchronous Context Managers
These are a fairly straightforward extensions of typical context managers.

An asynchronous context manager is one that can be used with an `async with` statement. An example is show here:

```python
async with FlowProvider(store_url) as provider:
    async with provider.open_read(flow_id, config=config) as reader:
        frames = await reader.read(720, count=480)

        # Do other things using reader
        ...

    # Do other things using provider
    ...

# Do something with frames
...
```

The difference between these and normal synchronous context managers is a simple one:

> [!IMPORTANT]
> The setup and teardown performed on entry and exit are performing by awaiting coroutines (instead of making a function call).

This means the code executed for entry and exit from the context can be asynchronous. It also means that `async with` can only be used in a context where asynchronous code is allowed.

## Asynchronous Iterators
An async [iteratable](../../../Coding/Python/Iterators%20and%20Generators.md) represents a source of data which can be looped over with an `async for` loop. Using one is straightforward:

```python
async for grain in reader.get_grains():
    # Do something with each grain object
    ...
```

The difference between this and a standard iterator is pretty simple: the method used to extract the next element from the asynchronous iterator is an asynchronous coroutine, and its output is awaited. 

In fact, `async for` is shorthand for a longer piece of code using `await` statements:

```python
async for a in async_iterable:
    await do_a_thing(a)

# Is equivalent to

it = async_iterable.__aiter__()
while True:
    try:
        a = await anext(it)
    except StopAsyncIteration:
        break

    await do_a_thing(a)
```

> [!INFO]
> There was a bit more info in the rest of the blog but I skipped it.