These notes are based on a combination of the [Python docs](https://docs.python.org/3/library/multiprocessing.html) and the [Educative course](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/g74259ZJM06).

Python's [multiprocessing](https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing) module supports three ways to [start a process](4%20-%20Processes#Creating%20a%20Process):

* Fork
* Spawn
* Forkserver

We describe each of these below. To select a method, the `set_start_method()` function can be used:

```python
import multiprocessing as mp
mp.set_start_method('spawn')
```

## Fork
The parent process forks the Python interpreter. The child process is effectively identical to the parent process. All resources of the parent are inherited by the child.

Note that forking a multithreaded process is problematic. For instance, after a `fork` of a multithreaded process, the child process is single threaded.[^fn1] So if another thread has a mutex locked, then the mutex will be locked in the child process but can **never be unlocked**.

Another drawback of `fork` is that often, we wish to avoid the exact cloning of a processes's data resources (data structures, file descriptors, database connections, etc). In this instance, `fork` isn't suitable.

## Spawn
The parent process starts a fresh Python interpreter process. The child process will only inherit those resources necessary to run the child process's `run()` method. In particular, unnecessary file descriptors and handles from the parent will not be inherited.

`spawn` is implemented as a `fork` followed by an `exec` syscall to replace itself with a fresh Python process. It then asks Python to load the target module and run the target callable. Due to this extra work done by the child process, `spawn` is slower than using [Fork](#Fork) or [Forkserver](#Forkserver). 

Another point to note is that if we pass arguments to the child process's callable target, the child **does** receive them. For instance in this code, the child process would receive both the global and local arguments:

```python
from multiprocessing import Process 
import multiprocessing 

global_arg = "this is a global arg" 

def process_task(garg, larg): 
	print(garg + " - " + larg) 
	
if __name__ == '__main__':
	multiprocessing.set_start_method('spawn') 
	local_arg = "this is a local arg" 
	process = Process(target=process_task, name="process-1", args=(global_arg, local_arg)) 
	process.start() 
	process.join()
```

Arguments to the child process are sent using [Pickle](https://docs.python.org/3/library/pickle.html). This is a Python module that implements serialisation of Python objects. Objects sent between processes are serialised by the sender and deserialised by the receiver.

## Forkserver
When a Python process runs `mp.set_start_method("forkserver")`, a new server process is started. From then on, whenever a new process is needed, the parent process connects to the server and requests that it fork a new process. 

Since the server process is single threaded, it is safe to use `fork`.

As with `spawn`, no unnecessary resources are inherited by the child process.

> [!NOTE]
> `forkserver` is less commonly used and not well documented.

## Semaphore Tracker
Just a final point - when we use `spawn` or `forkserver`, a "[semaphore](Mutex%20vs%20Semaphore.md) tracker" process gets created. On Unix systems, there is a limit to the number of semaphores that can exist. When a process creates a semaphore, the semaphore is linked to that process and may remain linked even if the process exits in an abnormal way..
The **semaphore tracker** process makes sure there are no leaking semaphores, i.e. it unlinks semaphores that were linked to processes that have already exited.


[^fn1]: Why this is, I'm not exactly sure. Something to do with POSIX compliance.





