What exactly is a package in Python? How are they structured?

These notes discuss these issues. They're based on the [Python docs](https://docs.python.org/3/tutorial/modules.html#:~:text=The%20__init__.py,on%20the%20module%20search%20path.), and [this webpage](https://pythongeeks.org/python-packages/).

```toc
```

## Basic Idea

A package is a way of organising related modules into a directory hierarchy. Essentially, it's a directory that contains multiple Python `.py` files, along with a special file `__init__.py`:

![](_attachments/Screenshot%202023-06-01%20at%2015.11.47.png)

The `__init__.py` file is run whenever the package is imported. For example, when running `import aladdin.ops.perf.clock_speed`, Python looks for an `__init__.py` file in each directory:

* `aladdin/` 
* `aladdin/ops/`
* `aladdin/ops/perf/`

If it finds `__init__.py`, it will execute the code within those files.

## `__main__.py`

This file is executed when a Python package or module directory is run directly as a script. For instance, if you were ran `python3 -m aladdin.ops`, Python would execute the `__main__.py` file within the `aladdin/ops/` directory.

## Importing * from a Package

Let's take this example package structure (stolen from the Python docs):

```
sound/                          Top-level package
      __init__.py               Initialize the sound package
      formats/                  Subpackage for format conversion
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
              ...
      effects/                  Subpackage for sound effects
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
      filters/                  Subpackage for filters
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
              ...
```

What happens when the user writes `from sound import *`?
We might think that this somehow imports all submodules in the package. But it does not.

Instead, the `import` statement uses the following convention: if a package’s `__init__.py` defines a list named `__all__`, it is taken to be the list of module names that should be imported when `from package import *` is encountered.

For example, the file `sound/effects/__init__.py` could contain the following code:

```python
__all__ = ["echo", "surround", "reverse"]
```

This would mean that `from sound.effects import *` would import the three named submodules of the `sound.effects` package.

If `__all__` is not defined, then `from sound.effects import *` does _not_ import all submodules from the package `sound.effects`; it only ensures that the package `sound.effects` has been imported (possibly running any code in `__init__.py`) and then imports whatever names are defined in the package.

> [!QUESTION]
> There is still plenty of stuff that I don't understand when it comes to Python imports.






