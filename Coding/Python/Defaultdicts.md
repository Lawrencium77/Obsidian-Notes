[These Educative notes](https://www.educative.io/answers/learning-about-defaultdict-in-python) were quite useful in understanding defaultdicts.

A `defaultdict` is a subclass of Python's in-built `dict` class. 
It populates the `__missing__()` method (see [below](#^b9beaf)) to provide a default value for a nonexistent key instead of raising a `KeyError`.

The default value can be specified during initialization of the `defaultdict` object and is provided by passing a callable object (e.g. a function) to the constructor.

When a key is accessed that doesn't exist in the `defaultdict`, the callable is executed and its return value becomes the value for that key in the dictionary.

Here is an example:

```python
from collections import defaultdict

# Create a defaultdict with default value 0
d = defaultdict(lambda: 0)

# Accessing a nonexistent key returns the default value (0 in this case)
print(d["non_existent_key"]) # Output: 0

# The key "non_existent_key" is now present in the defaultdict
d["non_existent_key"] += 1
print(d["non_existent_key"]) # Output: 1
```

It's also common to see the function `int`, `list` or `set` passed during initialization:

```python
from collections import defaultdict

# Create a defaultdict with default value of an empty list
d = defaultdict(list)

# Accessing a nonexistent key returns an empty list
print(d["non_existent_key"]) # Output: []

# Adding elements to the default value
d["non_existent_key"].append(1)
d["non_existent_key"].append(2)
print(d["non_existent_key"]) # Output: [1, 2]
```

### Side-Note on the `__missing__` method

^b9beaf

The `__missing__(self, key)` method defines the behaviour of a dictionary if you access a non-existent key. More specifically, Python's `__getitem__()` dictionary method internally calls the `__missing__()` method if the key doesn't exist.



