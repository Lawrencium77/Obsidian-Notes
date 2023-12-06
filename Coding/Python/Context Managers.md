[Context managers](https://book.pythontips.com/en/latest/context_managers.html) in Python are implemented as a class with two special methods: `__enter__()` and `__exit__()`. 

## Basics

Here's an example:

```python
class File(object):
    def __init__(self, file_name, method):
        self.file_obj = open(file_name, method)
    def __enter__(self):
        return self.file_obj
    def __exit__(self, type, value, traceback):
        print("Exception has been handled")
        self.file_obj.close()
        return True

with File('demo.txt', 'w') as opened_file:
    opened_file.undefined_function()
```

The `__enter__` method is executed at the beginning of the `with` block, and the `__exit__` method is executed at the end.

## Implementing a Context Manager as a Decorator
We can also create context managers using [Decorators](Decorators.md) and [generators](Iterators%20and%20Generators#Generators). Python has a `contextlib` module for this purpose. Here's an example:

```python
from contextlib import contextmanager

@contextmanager
def open_file(name):
    f = open(name, 'r')
    try:
        yield f
    finally:
        f.close()

with open_file('file.txt') as f:
   lines = f.readlines()
```

We can dissect what's going on by examining the [Python source code](https://github.com/python/cpython/blob/main/Lib/contextlib.py#L105).

`open_file` is passed as an `argument` to the context manager decorator, which is defined as:

```python
def contextmanager(func):
    @wraps(func)
    def helper(*args, **kwds):
        return _GeneratorContextManager(func, args, kwds)
    return helper
```

In simple terms - this means that calling `open_file(name)` instantiates and return a `_GeneratorContextManager` object. The `_GeneratorContextManager` class has this init method:

```python
def __init__(self, func, args, kwds):
	self.gen = func(*args, **kwds)
	self.func, self.args, self.kwds = func, args, kwds
	doc = getattr(func, "__doc__", None)
```

We see that it calls `open_file(name)` and sets it to the `self.gen` attribute.

It also has the following `__enter__` and `__exit__` methods:

```python
def __enter__(self):
	del self.args, self.kwds, self.func
	try:
		return next(self.gen)
	except StopIteration:
		raise RuntimeError("generator didn't yield") from None

def __exit__(self, typ, value, traceback):
	try:
		next(self.gen)
	except StopIteration:
		return False
```

Note that I simplified the `__exit__()` method here. But the idea is clear - `next()` is called on the generator function in both methods. In the `__exit__()` method, the inevitable `StopIteration` is caught and handled.



