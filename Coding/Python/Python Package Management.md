These notes summarise the reading I did on Python [package](Packages.md) management.
Some of these concepts apply to package management more broadly.

> [!INFO]
> I also made some separate notes on [Dependency Resolution in Pip](Dependency%20Resolution%20in%20Pip.md).

### Package Managers
A [package manager](https://en.wikipedia.org/wiki/Package_manager) is software that automates the process of installing, upgrading, configuring, and removing programs.

The most popular Python package manager is [Pip](https://en.wikipedia.org/wiki/Pip_(package_manager)).

Pip has a feature to manage full lists of packages and corresponding version numbers. This is made possible through a "requirements" file. 
This permits the re-creation of an entire group of packages in a separate environment.

This can be achieved by doing:

```
pip install -r requirements.txt
```

### "Build From Source"
In general, to **build** means to transform "source" files (edited by humans) into other files which are actually useful. This typically means converting files into an executable. For more info, see notes on [compilation systems](../../Computing/CSAPP/Chapter%201/It%20Pays%20to%20Understand%20How%20Compilation%20Systems%20Work.md).

It thus follows that the "from source" part of "build from source" is redundant; whenever a build process happens, it is by definition using doing so from source code.

The advantages of building from source are:
* You can customise the software
* You can stay up-to-date with the latest changes

The disadvantages are that this is more complex, and takes time.

The alternative to this is to download a **precompiled binary**. This is more straightforward, but sometimes lags behind the most up-to-date version of the source code.

### Pip Wheels
Pip is technically able to install either from source, or from precompiled binaries. These precompiled binaries are called **wheels**. 

When you use `pip` to install a package, it will first try to find a wheel that is compatible with your system. If it finds one, it will download and install the wheel, which typically results in a faster process compared to building the package from source.

If `pip` can't find a compatible wheel for a package, it will download the source distribution and attempt to build the package from source.

### Nightly Builds
For some projects the most up to date version of the source code is built on a nightly basis. These builds are "nightly builds".
