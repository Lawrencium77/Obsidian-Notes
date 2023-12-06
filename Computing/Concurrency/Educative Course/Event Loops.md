An [event loop](https://en.wikipedia.org/wiki/Event_loop) is a design pattern that's fundamental in asynchronous programming. It's at the heart of Python's [asyncio](https://docs.python.org/3/library/asyncio.html). The key idea is that an event loop repeatedly checks for and handles "events" or "messages". An event can be any kind of signal to the program, such as a mouse click, a key press, a network packet, a timer triggering, etc.

The event loop works in a cycle where it waits for an event to occur, and then dispatches the event to the appropriate handler, called a **callback**. A **callback** is a function that is passed as an argument to another function, with the intention that it will be invoked at a later time. The function that accepts the callback function is expected to execute it at a specific time. ^b5c521

If the callback is executed immediately, it's called a **synchronous**, or **blocking**, callback. If it happens at a later time, it's called **asynchronous**, or **non-blocking**.

Here is an example of a callback function in plain Python:

```python
def print_result(result):
    print(f"The result is {result}")

def calculate_sum(a, b, callback):
    sum = a + b
    callback(sum)

calculate_sum(5, 10, print_result)
```

This cycle - of waiting for an event, and then dispatching the event -  repeats indefinitely.

The key advantage of an event loop is that is allows for non-blocking I/O operations, despite Python being [single-threaded](Python%20GIL.md). In other words, the I/O operation can be run in the background and the program can continue to do other work.

> [!INFO]
> One of the most common use cases of event loops is in webservers. A webserver waits for a HTTP request to arrive and returns the matching resource. They run an event loop to receive web requests in a single thread. 

### Running the Event Loop
This section will show us how to create coroutines using asyncio, and how to run the event loop.

Consider this example:

```python
import asyncio
import time

async def do_something_important():
	await asyncio.sleep(10)

if __name__ == "__main__":
	asyncio.run(do_something_important())
```

Let's explain it!

First, the `async def` statement declares an [coroutine](../../../Coding/Python/Coroutines%20and%20Subroutines.md). Calling `do_something_important()` returns a coroutine object. 

Second, all Python coroutines must contain the `await` keyword. It is used to pause the coroutine's execution until the awaited task completes. This allows other tasks in the event loop to run.

Third, `asyncio.sleep()` is *another* coroutine that blocks for the specified number of seconds. This means we're calling a coroutine from within a coroutine.

Finally, `asyncio.run()` is a high-level function call that creates a new event loop, runs the passed coroutine, and then closes the event loop.

In this context, it's helpful to think of the event loop as an entity that schedules and runs tasks, which are instances of coroutines.

#### Running Multiple Coroutines
Of course, the real strength of the event loop is to handle multiple coroutines [concurrently](Concurrency%20vs%20Parallelism.md):

```python
import asyncio

# Define a coroutine
async def say_after(delay, msg):
    await asyncio.sleep(delay)
    print(msg)

# Define a main coroutine that runs other coroutines
async def main():
    print(f"Started at {time.strftime('%X')}")

    # Run the coroutines concurrently using asyncio.gather
    await asyncio.gather(
        say_after(3, 'Hello'),
        say_after(3, 'World'),
    )

    print(f"Finished at {time.strftime('%X')}")

# Run the main coroutine
asyncio.run(main())
```

The `gather()` call means that we run each of the coroutines concurrently. The key point is that this program would only take 3 seconds to run. 

#### Chaining Coroutines
One important use of coroutines it to chain them to process data pipelines. The idea is that an input passes, through the first coroutine, which may perform some actions on the data. It is then passed to the second coroutine, and so on.

Here's an example:

```python
async def coro3(k): 
	return k + 3 
	
async def coro2(j): 
	j = j * j 
	res = await coro3(j) 
	return res 
	
async def coro1(): 
	i = 0 
	while i < 100: 
		res = await coro2(i) 
		print("f({0}) = {1}".format(i, res)) 
		i += 1 
		
if __name__ == "__main__": 
	# Calculate x^2+3 for the first 100 ints 
	cr = coro1() 
	loop = asyncio.get_event_loop()
	loop.run_until_complete(cr)
```





