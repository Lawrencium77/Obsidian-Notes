These notes are based upon [pip's docs](https://pip.pypa.io/en/stable/topics/dependency-resolution/).

When a user does a `pip install` (e.g. `pip install tea`), `pip` needs to work out the [package's](Python%20Package%20Management.md) dependencies (e.g. `spoon`, `hot-water`, `tea-leaves`, etc), and what versions of each it should install.

The process of determining which dependency to install is called **dependency resolution**.

```toc
```
## The Dependency Resolution Problem
The process of finding a set of packages to install, given a set of dependencies between them, is an [NP-hard](https://en.wikipedia.org/wiki/NP-hardness) problem[^fn2]. This means that the process scales *really* badly with problem size.

In practice, this means there will be situations where `pip` cannot determine what to install in a reasonable length of time. 

Moreover, it can be simply impossible to find a combination of package versions that do not conflict. This is called [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell).

> [!INFO]
> `pipdeptree` is a command line utility for displaying the installed python packages in form of a dependency tree. See its [PyPI page](https://pypi.org/project/pipdeptree/) for more.

## Python Specific Issues
When handling dependency resolution in Python, there is a severe limitation: dependency metadata is only available by **downloading**[^fn4] the package file, and extracting the data from it. In other words, we do not know the full details of the problem up front.

Work is ongoing to try to make metadata more readily available at a lower cost. But at the time of writing (24/05/23), this has not been completed. [^fn3]

Since downloading packages is costly, `pip` cannot pre-compute a full dependency tree. This means we're unable to use a number of techniques for solving dependency resolution, and must instead use [Backtracking](#Backtracking).


## Backtracking
During dependency resolution, `pip` needs to make assumptions about the package versions it needs to install. Later, it checks these assumptions were not incorrect. When `pip` finds that an assumption it made was incorrect, it has to **backtrack**. This means discarding some of the work that has already been done.

Consider an example:  `pip install tea`. The package `tea` declares a dependency on `hot-water`, `spoon`, and `cup`.

`pip` first picks the most recent version of `tea` and gets the list of dependencies on that version of `tea`. It then repeats the process for those packages, picking the most recent version of `spoon` and `cup`.
Suppose `pip` notices that the version of `cup` is not compatible with the version of `spoon` it has chosen[^fn1]. In this case, it will *backtrack* and try to use another version of `cup`. If it is successful, it will continue onto the next package. Otherwise, it will continue to backtrack on `cup` until it finds a version that is compatible with all other packages.

This looks like:

```
pip install tea
Collecting tea
  Downloading tea-1.9.8-py2.py3-none-any.whl (346 kB)
     |████████████████████████████████| 346 kB 10.4 MB/s
Collecting spoon==2.27.0
  Downloading spoon-2.27.0-py2.py3-none-any.whl (312 kB)
     |████████████████████████████████| 312 kB 19.2 MB/s
Collecting cup>=1.6.0
  Downloading cup-3.22.0-py2.py3-none-any.whl (397 kB)
     |████████████████████████████████| 397 kB 28.2 MB/s
INFO: pip is looking at multiple versions of this package to determine
which version is compatible with other requirements.
This could take a while.
  Downloading cup-3.21.0-py2.py3-none-any.whl (395 kB)
     |████████████████████████████████| 395 kB 27.0 MB/s
  Downloading cup-3.20.0-py2.py3-none-any.whl (394 kB)
     |████████████████████████████████| 394 kB 24.4 MB/s
  Downloading cup-3.19.1-py2.py3-none-any.whl (394 kB)
     |████████████████████████████████| 394 kB 21.3 MB/s
  Downloading cup-3.19.0-py2.py3-none-any.whl (394 kB)
     |████████████████████████████████| 394 kB 26.2 MB/s
  Downloading cup-3.18.0-py2.py3-none-any.whl (393 kB)
     |████████████████████████████████| 393 kB 22.1 MB/s
  Downloading cup-3.17.0-py2.py3-none-any.whl (382 kB)
     |████████████████████████████████| 382 kB 23.8 MB/s
  Downloading cup-3.16.0-py2.py3-none-any.whl (376 kB)
     |████████████████████████████████| 376 kB 27.5 MB/s
  Downloading cup-3.15.1-py2.py3-none-any.whl (385 kB)
     |████████████████████████████████| 385 kB 30.4 MB/s
INFO: pip is looking at multiple versions of this package to determine
which version is compatible with other requirements.
This could take a while.
  Downloading cup-3.15.0-py2.py3-none-any.whl (378 kB)
     |████████████████████████████████| 378 kB 21.4 MB/s
  Downloading cup-3.14.0-py2.py3-none-any.whl (372 kB)
     |████████████████████████████████| 372 kB 21.1 MB/s
```

Backtracking can take a while. The amount of time depends on the package size, the number of versions `pip` must try, and various other factors.

Here's a simplified summary of this algorithm:

> 1. Start by choosing the latest version of the package you want to install.
> 2. Fetch the metadata for this package version and determine its dependencies.
> 3. For each dependency, choose the latest version that satisfies the version constraint specified by the parent package.
> 4. Continue this process recursively for each dependency's dependencies.
> 5. If you get to a point where a chosen version conflicts with another package's requirements, then we backtrack: discard the current choice and try the next latest version of the package.
> 6. Continue until you find a set of package versions that satisfies all constraints, or until you've exhausted all options and determined that no such set exists.

> [!INFO]
> Backtracking is a type of exhaustive search. It can also be interpreted as a depth-first search. If there exists a valid solution, it will find it. See [these notes](https://web.stanford.edu/class/archive/cs/cs106b/cs106b.1188/lectures/Lecture11/Lecture11.pdf) for more info.

> [!QUESTION]
> I haven't *fully* grokked the backtracking algorithm. But I get the general idea.


## Installation Order
This section is based upon [this page](https://pip.pypa.io/en/stable/cli/pip_install/#installation-order) .

`pip` **installs** dependencies before dependents. This is the only commitment that `pip` makes related to order.  In the event of a circular dependency, the current implementation has it such that the first member of the cycle is installed last.

For instance, if `quux` depends on `foo` which depends on `bar` which depends on `baz`, which depends on `foo`:

![](_attachments/Screenshot%202023-05-24%20at%2020.41.09.png)

Then `pip` installs packages in the following order:

```
python -m pip install quux
...
Installing collected packages baz, bar, foo, quux
```

The decision to install in this way is based on the principle that installations should proceed in a way that leaves the environment usable at each step. This has two main practical benefits:

1. Concurrent use of the environment during the install is more likely to work.
2. A failed install is less likely to leave a broken environment. 

## Summary
To be clear:`pip` downloads packages before it installs them. It needs to download a package to inspect its dependencies. This allows it to build a dependency graph. 

To summarise, when you run  `pip install package`, the following things happen:

1. `pip` fetches metadata for `package` from PyPI.
2. From this metadata, `pip` learns about the dependencies of `package`.
3. `pip` repeats this process for each of the dependencies, and their dependencies, and so on, recursively.
4. This builds a tree of dependencies.
5. Based on this tree, `pip` then determines an installation order that respects the "dependencies before dependents" rule. This ordering helps avoid any potential issues with unsatisfied dependencies.
6. Finally, `pip` proceeds to actually install the packages, in the order determined in step 5.




[^fn1]: One possible cause for this is that `spoon` and `cup` have conflicting dependencies. Alternatively, `cup` could be dependent upon `spoon` (or vice versa) in a way that is not compatible.
[^fn2]: If we assume that P ≠ NP, then NP-hard problems cannot be solved in polynomial time.
[^fn3]:  Current Python packaging standards encourage specifying all metadata and dependencies in a static file. But other factors, such as dynamic dependencies, make achieving this harder. Dynamic dependencies mean that environmental factors, such as your OS, can affect the dependencies of a package
[^fn4]: (But not installing)