These are some brief notes on typehints in Python.
They are based upon:

* [This](https://realpython.com/lessons/type-hinting/) blog, which gives a really basic outline.
* The [python docs](https://docs.python.org/3/library/typing.html).
* [This](https://towardsdatascience.com/12-beginner-concepts-about-type-hints-to-improve-your-python-code-90f1ba0ac49) TDS blog has example syntax.

## The Basic Idea
The purpose of typehints is to add type information to our code. Below is a simple example that shows a function with typehints. We can annotate the arguments and return value:

```python
def greet(name: str = None) -> str:
	return "Hello," + name
```

> [!INFO]
> Typehints are **not** enforced at runtime. They instead can be used by third party tools such as type checkers, IDEs, linters, etc.

## More Examples

### Perform Static Type Checking with mypy
[mypy](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html) is a tool that does static type checking.
We run it from the command line, for instance:

```shell
mypy script.py
```

It raises an error when an argument is passed with an incorrect type.

### Lists
To build more advanced types, we use Python's `typing` module. For instance, to support a list of floats we'd do the following:

```python
from typing import List

def my_dummy_function(vector: List[float]):
	return sum(vector)
```

### Dictionaries
Same goes for dictionaries. When using `Typing`, the first argument is the key type, and the second is the value type:

```python
from typing import Dict

my_dict_type = Dict[str, str]
```
















[mypy docs](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html) have some good stuff







