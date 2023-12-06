
## Iterables
**Iterables** are objects that are capable of returning their members one at a time. Basically, anything you can loop over (usually with a `for` loop) is an iterable.

Examples include lists, strings, dictionaries, tuples and text files:

```python
for i in [1,2,3,4]:
	print(i)

for c in "python":
	print(c)
	
for k in {"x": 1, "y": 2}:
	print(k)
```
There are many functions that consume these iterables, e.g:
```python
",".join(["a", "b", "c"])

list("python")
```

All iterables have an `__iter__` method. When we call a `for` loop on an iterable, in the background this calls the `__iter__()` method on the object. This returns an **iterator** that it loops over. 

### The Iteration Protocol
The built-in Python function `iter` takes an iterable object and returns an **iterator**:
```python
>>> x = iter([1, 2, 3])
>>> x
<listiterator object at 0x1004ca850>
>>> next(x)
1
>>> next(x)
2
>>> next(x)
3
>>> next(x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```
A key difference between an iterator and an iterable is that an iterator has a *state*, such that it 'remembers' where it is after each iteration. 

Another key difference is that an iterator 'knows' how to access its next element: each time we call the `next` method on the iterator, it gives us the next element. If there are no more elements, it raises `StopIteration`.

Many built-in functions accept iterators as arguments:
```py
>>> list(x)
[1,2,3]
>>> sum(x)
6
```

We can implement our own iterators using classes. Here is an iterator that works like the built-in `range` function:

```python
class MyRange:

	def __init__(self, n):
		self.i = 0
		self.n = n
		
	# iterator returns itself when passed to the iter function
	def __iter__(self):
		return self
		
	def __next__(self):
		if self.i < self.n:
			i = self.i
			self.i += 1
			return i 
		else:
			raise StopIteration()
```
When defining an iterator, we must provide  `__iter__` and `__next__` methods. The `__iter__` method is what makes an object iterable. 
As mentioned before, all iterators have a state. In the code above, this state is defined by `self.i`. 

Behind the scenes, using the *iter* on an iterable calls the `__iter__` method on the given object. I.e. the following lines of code return the same output
```python
x = [1,2,3]
iterator1 = x.__iter__()
iterator2 = iter(x)
```

The key point is that the return value of `__iter__` must be an iterator. The class we have defined is an iterator class, so it is sufficient to simply return `self` in the `__iter__` method.

When we call `next()` on an iterator, Python calls the `__next__()` method.

## Generators
Generators simplify the creation of iterators.  A **generator** can be thought of as a function that produces a sequence of results instead of a single value.

```python
def yrange(n):
	i = 0
	while i < n:
		yield i
		i += 1
```
Note that generators are defined as functions, not classes. Each time the `yield` statement is executed, the function generates a new value.
```py
>>> y = yrange(3)
>>> y
<generator object yrange at 0x401f30>
>>> next(y)
0
>>> next(y)
1
>>> next(y)
2
>>> next(y)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```
A generator is also an iterator, but you don't have to worry about the iterator protocol. The `__iter__` and `__next__` methods are created automatically.

The word "generator" is confusingly used to mean both the function that generates, and what it generates. In these notes, we will use "generator" to mean the generated object and "generator function" to mean that function that generates it.

Let's think about how this is working internally.

When a generator function is called, it returns a generator object without even beginning the execution of the function. When the `next` method is called for the first time, the function starts executing until it reaches the `yield` statement. The yielded value is returned by the `next` call.

The following example demonstrates the interplay between `yield` and call to `__next__` on a generator object:
```py
>>> def foo():
...     print("begin")
...     for i in range(3):
...         print("before yield", i)
...         yield i
...         print("after yield", i)
...     print("end")
...
>>> f = foo()
>>> next(f)
begin
before yield 0
0
>>> next(f)
after yield 0
before yield 1
1
>>> next(f)
after yield 1
before yield 2
2
>>> next(f)
after yield 2
end
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```
I will repeat this point: calling `next()` on the generator object causes the generator function to execute until it reaches `yield`, at which point it stops.

Iterators/generators can be used to generate sequences of arbitrary length, e.g:

```python
def my_range(start):
	current = start
	while True:
		yield current
		current += 1
```
In general, generators/iterators can be extremely useful when making programs **memory efficient**. Instead of having to store large lists, we can simply define a function that generates the values we need.
## Generator Expressions
These are a generator version of list comprehensions. They look like list comprehensions, but return a generator instead of a list:

```py
>>> a = (x*x for x in range(10))
>>> a
<generator object <genexpr> at 0x401f08>
>>> sum(a)
285
```
