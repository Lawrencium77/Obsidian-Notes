These notes are based on a combination of [this](https://chat.openai.com/share/92ded55b-a1c6-491a-bdb1-72fee941a02a) ChatGPT convo, [this](https://makefiletutorial.com/#getting-started) blog and [these](https://www.educative.io/courses/learn-c-from-scratch/7X5lvZJYjmr) Educative notes.

> [!INFO]
> **Important context**: whilst Makefiles can be used to automate the build process, they're now often used to automate simpler, high-level tasks.

## What are Makefiles?
Makefiles are used my the `make` utility. `make` is a build automation tool that automatically builds executables and libraries from source code by reading from Makefiles. It automatically determines which pieces of a program need to be recompiled, and issues commands to recompile them.

So, in short, they are primarily used to automate code compilation. But they can also be used to automate other tasks, like copying files, running linters, and running unittests.

A common pattern is for a project to use CMake as its build system generator, and use a Makefile for simpler tasks. These corresponding Makefiles are often very simple and are not used to compile the software. This is exactly how they're used in **Aladdin**.


## How Do They Relate to CMake?
CMake is a more advanced and flexible version of Make.

## How to Use Them?
We'll cover only the basics here.

A Makefile consists of a set of rules. Each rule has the format:

```
target: prerequisites
   recipe
```

The **target** is either the name of the file that is to be generated, or the name of an action to carry out.

The **prerequisites** are files that the target depends upon. They are also known as **dependencies**. Rules can serve as prerequisites for other rules.

The **recipe** is an action that `make` carries out. It's essentially just a series of commands that are run.

Here's an example of a simple Makefile that could be used to compile a C++ program:

```
program: main.o utility.o
    g++ main.o utility.o -o program

main.o: main.cpp
    g++ -c main.cpp

utility.o: utility.cpp
    g++ -c utility.cpp

clean:
    rm *.o program
```

#### Phony Targets
In the above example example, if you happen to have a file named `clean`, this target won't run. This is not what we want.

To resolve this, **phony** targets are targets that are not associated with a file. In other words, `.PHONY` to a target will prevent Make from confusing the phony target with a file name.

For example, in this Makefile:

```
.PHONY: all clean

all: clean
	<some command>

clean:
	rm -rf build
```

we've two targets - `all` and `clean` - both of which are phony.

