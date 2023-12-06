These notes are based on the [Python docs](https://docs.python.org/3/tutorial/modules.html#compiled-python-files).

When running Python code, we often see a `__pycache__` directory appear. Why is this? 

To speed up loading [modules](../Modules%20and%20Scripts.md), Python caches the compiled version of each module in the `__pycache__` directory. It does so under the name `module.version.pyc`, where the version encodes the format of the compiled file; it generally contains the Python version number.

When a module is imported, Python first checks the `__pycache__` directory for a compiled version. If it exists, and the source file hasn't been modified since the compiled file was created, Python will use the compiled version instead of re-interpreting the source code. If the source code has been modified since the compiled version was created, or if there isn't a compiled version, Python will interpret the source code and create a new compiled version for future use.

To be clear: a program doesn't *run* any faster when read from a `.pyc` file. The only things that's faster about `.pyc` files is the speed at which they're loaded.

