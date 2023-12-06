Suppose we run the following code:

```python
import sys
print(sys.path)
```

An example output is as follows:

```bash]
> python3 aladdin/body/tmp.py
['/home/lawrencea/git/aladdin/aladdin/body', '/usr/lib/python36.zip', '/usr/lib/python3.6', '/usr/lib/python3.6/lib-dynload', '/usr/local/lib/python3.6/dist-packages', '/usr/lib/python3/dist-packages']

```

What does this actually mean?

As described in the [Python docs](https://docs.python.org/3/library/sys.html#sys.path),  the output is a list of strings that specifies the interpreter's search path for Python modules. It's useful to understand how this is populated.

By default, a path is prepended to `sys.path`. Exactly *what* this path is depends on whether we're running a [module, script, or REPL](Modules%20and%20Scripts.md):

> * `python -m module` command line: prepend the current working directory
> * `python script.py` command line: prepend the script's directory.
> * `python -c code` and `python` (REPL) command lines: prepend an empty string, which means the current working directory.

The next items in `sys.path` are those contained in the environment variable `PYTHONPATH`. We can set `PYTHONPATH` to whatever we like.

Finally, `sys.path` contains the installation-dependent default directory. This is controlled by the `site` module. The [site module](https://docs.python.org/2/library/site.html#module-site) is automatically imported when you start Python. Exactly how it manipulates `sys.path` is a bit involved, but you can read about it in the Python docs.
For now, suffice to say that `site-packages` is a directory within a Python installation where third-party libraries are installed. [pip](Dependency%20Resolution%20in%20Pip.md) installs packages here. Its exact location depends upon your OS and the version of Python that you're using. For example, in a typical Python installation on a Unix-like OS, the `site-packages` directory might be located at `/usr/local/lib/python3.8/site-packages`. On a Windows system, it might be located at `C:\Python38\Lib\site-packages`.



