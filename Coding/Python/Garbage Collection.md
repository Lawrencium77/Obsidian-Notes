These notes are largely based on [this blog](https://rushter.com/blog/python-garbage-collector/). 

```toc
```

## Garbage Collection Algorithms
The aim of garbage collection is to reclaim memory which was allocated by a program but is no longer referenced; such memory is called **[garbage](https://en.wikipedia.org/wiki/Garbage_(computer_science))**.

In Python, [everything is an object](del%20Keyword#^6c564d). Knowing when to allocate them is easy. But automatic deallocation is tricky. Python needs to know when your object is no longer needed. 

Garbage collection algorithms track which objects can be deallocated and pick an optimal time to deallocate them. Python's garbage collector has two components: 

1. The **reference counting** collector
2. The generational **garbage collector**, known as the [gc](https://docs.python.org/3.6/library/gc.html) module

The **reference counting** algorithm is efficient & straightforward but cannot detect reference cycles. That's why Python has a supplemental algorithm called **generational cycle GC**. It deals with reference cycles only.

The reference counting module is fundamental to Python and can't be disabled. The cycle GC is optional and can be triggered manually.

The rest of these notes describe both algorithms in more detail. 

## Reference Counting

> [!INFO]
> "Reference" is another word for "pointer".

Every variable in Python is reference to an object and not the actual value itself. A single object can have many references.

This code creates two references to a single object:

```python
a = [1,2,3]
b = a
```

We can verify this using Python's `id()` function:

```python
id(a) == id(b) # True
```

To keep track of references, every object has an extra field called **reference count** that is increased or decreased when a pointer to the object is created or deleted.

### Examples Where the Reference Count Increases

* Assignment operator
* Argument passing
* Appending an object to a list (object's reference count is increased)

If the reference count reaches zero, Python automatically calls the object-specific memory deallocation function. If this object contains references to other objects, then their reference count is automatically decremented too. Thus other objects may be deallocated in turn.

[Global variables](Scope%20in%20Python/Scope%20Types.md) live until the end of the Python process. To keep them alive, all globals are stored inside a dictionary that you can access with the `globals()` function.

As for [local variables](Scope%20in%20Python/Scope%20Types.md): when the Python interpreter exits from a block, it destroys local variables and their references. It's important to understand that while your program remains in a local scope, the Python interpreter assumes that all variables inside are in use. To remove something from memory, you need to either assign a new value to a variable or exit from a block of code (e.g. a function).

We can check the reference count to an object with the `sys.getrefcount` function:

```python
import sys

foo = []
# 2 references: 1 from foo, and 1 from getrefcount
print(sys.getrefcount(foo))

def bar(a):
	# 4 references:
	# foo, function argument, getrefcount and Python's function stack
	print(sys.getroundcount(a)) 

bar(foo)
# 2 references since the function scope is destroyed
print(sys.getrefcount(foo))
```

Sometimes, you need to remove a global or local variable prematurely. To do so, you can use the [del statement](del%20Keyword.md) that removes a variable and its pointer (but not the object itself). This is useful in **Jupyter notebooks** because all cell variables use the global scope.

## Generational Garbage Collector
Why do we need an additional garbage collector when we have reference counting?

Classical reference counting has a fundamental problem - it cannot detect reference cycles. A reference cycle occurs when one or more objects are referencing each other:

![|500](_attachments/Screenshot%202023-02-12%20at%2013.18.29.png)

Here's the first example in code form:

```python
import gc

# We use ctypes to access our unreachable objects by memory address
# Don't worry about how it works
class PyObject(ctypes.Structure):
    _fields_ = [("refcnt", ctypes.c_long)]

gc.disable()  # Disable generational gc

object_1 = {}
object_2 = {}
object_1['obj2'] = object_2
object_2['obj1'] = object_1

obj_address = id(object_1)

# Destroy references
del object_1, object_2

# Uncomment if you want to manually run garbage collection process 
# gc.collect()

# Check the reference count
print(PyObject.from_address(obj_address).refcnt) # 1
```

In this example, the `del` statement removes the references to our objects (i.e. decreases reference count by 1). After Python executes the `del` statement, the objects are no longer accessible from the Python code. However, they're still sitting in memory.
This happens because they are referencing each other, and the reference count of each object is 1.

The `gc` module is responsible for resolving this issue.

Reference cycles can only occur in **container objects**.

> [!INFO]
> In Python, **containers** are objects that hold an arbitrary number of other objects. Generally, containers provide a way to access the contained objects and to iterate over them.
> Examples include  lists, dictionaries, tuples, and classes. 


### When Does the Generational GC Trigger?
Unlike reference counting, generational GC doesn't work in real-time. It runs periodically. To reduce the frequency of GC calls, Python uses various heuristics.

For more info, see the [original blog post](https://rushter.com/blog/python-garbage-collector/#:~:text=Reference%20counting%20is%20a%20simple,to%20the%20right%2Dhand%20side.).

### How are Reference Cycles Found?
It's hard to explain this in a few paragraphs. The original post gives some resources to read about this.



















