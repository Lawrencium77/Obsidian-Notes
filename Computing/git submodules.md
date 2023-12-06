The [git docs](https://git-scm.com/book/en/v2/Git-Tools-Submodules) do an excellent job of explaining submodules. I just made notes here so that I can get down the key points, and Ankify them.

## Submodules
It often happens that while working on one project, you need to use another project from within it. Perhaps it's a third-party library, or another project of your own. 

Git addresses this issue with **submodules**. These allow you to keep a Git repo as a subdirectory of another repo. This lets you clone another repo into your project and keep your commits separate.

> [!INFO]
> I came across this when using Pybind to create Python bindings for a C++ tensor processing library I was writing.


## Setting up a submodule
You can add an existing git repo as a submodule of the repo you're currently working on by doing:

```
git submodule add <url>
```

This creates a `.gitmodules` file. This is a config file that stores the mapping between the project's URL and the local subdirectory you've pulled it into. It looks like:


```
[submodule "<submodule name>"]
	path = <submodule name>
	url = <submodule URL>
```

This file is version-controlled, like your `.gitignore` file.

## Cloning a Project with Submodules
How to clone a project with submodules in it? When you clone such a project, by default you get the directories that contain submodules, but none of the files within them yet.

After running `git clone <URL>`, you must also run:

* `git submodule init` - this initialises the local config file
* `git submodule update` - this fetches all the data from that project, and checks out the appropriate commit listed in the superproject.

We can combine all these commands into one by passing the `--recurse-submodules` to the `git clone` command:

```
git clone --recurse_submodules <URL>
```

> [!INFO]
> There is a bunch more stuff in the docs on pulling upstream changes from the submodule remote, project remote, etc. This is interesting, but I'll come back round to it if I ever have the need!




