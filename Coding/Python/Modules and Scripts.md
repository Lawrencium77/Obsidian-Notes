A Python [module](https://docs.python.org/3/tutorial/modules.html) is a file containing definitions and statements. It's designed to be imported and used by other Python scripts or modules.

A Python **script** is a file that's intended to be run directly. It usually has a `__name__` attribute set to `__main__` when it is run, which allows you to include code that should only be executed when the script is run directly.

A Python file can be both a module and a script, depending on how it's used. That's why it's common to see  `if __name__ == __main__` in Python files.

`__name__`  is a special built-in variable in Python that holds the name of the current module or script. When a Python file is imported as a module, `__name__` is set to the module's name, which is the same as the filename without the `.py` extension.
When a Python script is run directly, `__name__` is set to `__main__`. 


#### Interface Options to Python command
When invoking `python` from the command line, there are several options we can specify.

Running `python` puts us in a Python REPL.

Running `python -c <command>` executes the specified Python code.

Running `python <script>` executes the Python code contained in *script*.

Running `python -m <module>` searches for the named module and executes its contents as the `__main__` module. In other words, it executes the given Python module as a script. Since the argument is a *module* name, you must not give a file extension. 
I am not 100% sure on why you'd ever actually do this, instead of just calling `python <script>`. I think that one reason is that it can be useful when running a module that is installed in your system, without needing to know its exact location.



