In Python's `multiprocessing` module, there are two mechanisms for interprocess communication (IPC):

* Queues
* Pipes

These notes cover how these work. They're based on the [Python docs](https://docs.python.org/3/library/multiprocessing.html#pipes-and-queues) and [Educative](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/YVOkk3xyP0K).

## Queue
The `Queue` class is thread and process safe. It provides FIFO data structure for use as a shared buffer between processes, as well as a protocol for sending and receiving objects. Note that a `Queue` is implemented using a `Pipe` and a few [locks/semaphores](Mutex%20vs%20Semaphore.md).

It can be used by multiple consumers and producers.

Here is an example using a `Queue` to solve the [producer/consumer problem](Mutex%20vs%20Semaphore.md#^990b6a). Note the methods `put()` and `get()`, which add and retrieve items to/from the `Queue`:

```python
from multiprocessing import Process, Queue, current_process
import time
import random

def producer(queue):
    for item in range(10):
        queue.put(item)
        print(f'Process {current_process().name} added item {item} to the queue')
        time.sleep(1)

def consumer(queue):
    counter = 0
    max_misses = 3
    while True:
        if not queue.empty():
            item = queue.get()
            print(f'Process {current_process().name} retrieved item {item} from the queue')
            time.sleep(2)
        else:
            counter += 1
            if counter > max_misses:
                break

if __name__ == "__main__":
    queue = Queue()
    producer_process = Process(target=producer, args=(queue,), name='Producer')
    consumer_process = Process(target=consumer, args=(queue,), name='Consumer')

    producer_process.start()
    consumer_process.start()

    producer_process.join()
    consumer_process.join()

```

## Pipe
`Pipe` objects provide either a one-way or two-way connection between **two** processes (they can't be used for $\gt 2$ processes).  Whatever is written to one end of the pipe can be retrieved from the other end.

The `Pipe` API is:

```python
multiprocessing.Pipe(duplex=False)
```

This returns a pair `(conn1, conn2)` of `Connection` objects representing the ends of the pipe. If `duplex=True` then the pipe is bidirectional. 

Just like `Queue`s, `Pipe`s are essentially a buffer for data transfer between processes. 

> [!INFO]
> Note that data in a pipe may become corrupted if two processes (or threads) try to read from or write to the **same** end of the pipe at the same time. This is why they're not meant to be used with $\gt 2$ processes. There is no risk of corruption from processes using different ends of the pipe at the same time.

When using `Pipe`s, it's useful to know that `Connection` objects use the `send` and `recv` methods to send and receive objects. Here is a code example:

```python
from multiprocessing import Process, Pipe 
import time 

def child_process(conn): 
	for i in range(0, 10): 
		conn.send("hello " + str(i + 1)) 
	conn.close() 
	
if __name__ == '__main__': 
	parent_conn, child_conn = Pipe() 
	p = Process(target=child_process, args=(child_conn,)) 
	p.start() 
	time.sleep(3) 
	
	for _ in range(0, 10): 
		msg = parent_conn.recv() 
		print(msg) 
		
	parent_conn.close() 
	p.join()
```


## Non-Blocking Method Calls
The `Queue` class's `get()` and `put()` methods have an optional argument, `block`. This determines the behaviour of the method when the operation cannot complete immediately:

`block=True` (default):

* If `block=True` for `put()` and the queue is full, the method will block (i.e., wait) until a queue slot becomes available.
* If `block=True` for `get()` and the queue is empty, the method will block until an item becomes available.

**`block=False`:**

- If `block=False` for `put()` and the queue is full, the method will immediately raise a `queue.Full` exception.
- If `block=False` for `get()` and the queue is empty, the method will immediately raise a `queue.Empty` exception.

A `Connection` object's `recv()` method doesn't support this directly, but the `poll()` method can be used to check if there is anything to receive before calling `recv()`. This achieves the same thing.