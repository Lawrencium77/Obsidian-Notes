This is a single document which describes all of my reading on CMake. I might refactor it at a later point.

#### Sources
https://vnav.mit.edu/labs/lab1/cmake.html
https://cliutils.gitlab.io/modern-cmake/chapters/basics.html

```toc
```

## High-Level Motivation
> [!INFO]
> [Build Systems](https://en.wikipedia.org/wiki/List_of_build_automation_software) refer to any build automation software. Examples include Make, CMake, and Ninja. CMake is an example of a *generator* tool - one which doesn't build directly, but rather generates files to be used by *native* build tools.

For small-scale projects, it is sufficient to use command-line commands to directly compile programs. However, for large C++ codebases, we need a better and more automatic tool to compile. In addition, different platforms might require different compilers.

This is where build systems are useful. Users of such systems write some configuration files, from which the build system will generate native [makefiles](../Makefiles.md) for compilation.

CMake is one of such systems. With CMake, we write `CMakeLists.txt` files that specify where to find source code and header files. When then call CMake to generate makefiles.

## The Basics
### Minimum Version
Most `CMakeLists.txt` files start with the following line:

```C
cmake_minimum_required(VERSION 3.1)
```

`cmake_minimum_required` is case insensitive, but is usually lower case. `VERSION` is a special keyword for this function. And the value of the version follows the keyword.

This function sets the minimum required version of CMake for a project. It also updates the **policy** settings. A CMake [policy](https://cmake.org/cmake/help/latest/command/cmake_policy.html#version) is described below:

> [!INFO]
> As CMake evolves, it's sometimes necessary to change existing behaviour to fix bugs or improve current features. The CMake policy mechanism is intended to keep existing projects building correctly, even as new versions of CMake introduce changes in behaviour.  Each policy is given an identifier of the form `CMP<NNNN>`. When CMake needs to know which behaviour to use, it checks for a setting specified by the project. 

This means that if you set `minimum_required` to `VERSION 3.1`, the policy is set to be compatible with version `3.1`. 

Starting with CMake 3.12, `cmake_minimum_required` supports a range, such as `VERSION 3.1...3.16`. This means you support as low as `3.1` but have also tested with new policy settings up to `3.15`. 

### Setting a Project
Every top-level CMake file will have the next line:

```C
project(MyProject)
```

This is used to set the project name and metadata. Here's an example that sets the project version, description, and supported languages:

```C
project(MyProject VERSION 1.0
                  DESCRIPTION "Very nice project"
                  LANGUAGES CXX)
```

There's nothing special about the project name. No [targets](#Targets) are added at this point.

### Making an Executable
The [add_executable](https://cmake.org/cmake/help/latest/command/add_executable.html) command is used to specify that an executable should be built from the given source files.

```C
add_executable(one two.cpp three.h)
```

There's a few things to understand here. `one` is the name of the executable file generated. `two.cpp` is the source file that will be compiled to create the executable. CMake will invoke the compiler to do this. `three.h` is not strictly necessary to include, as header files are typically included within the source files. CMake is smart, and will only compile source code files. However, listing it here can be useful for certain IDEs to know that this header file is associated with the target. 

### Making a Library
Similarly, the [add_library](https://cmake.org/cmake/help/latest/command/add_library.html) command is used to create a library from a set of source files:

```C
add_library(one STATIC two.cpp three.h)
```

Again, `one` is the name of the library to be created. Next, we can pick the library type from one of:

```
STATIC
SHARED
MODULE
```

`STATIC` creates a static library, `SHARED` creates a shared (aka dynamically linked) library, and `MODULE` creates a dynamically loaded library. `two.cpp` and `three.h` have the same roles as with the [add_executable](#Making%20an%20Executable) command.

### Targets Are Your Friend
Targets are exactly as they're described in [makefiles](Makefiles#How%20to%20Use%20Them?): the target is the name of the executable or library that's created. Now that we know how to create them, we can also add various bits of information to them. For example, this command:

```C
target_include_directories(one PUBLIC include)
```

specifies a list of directories to be added to the include path for a specific target during the build. These directories contain header files that the compiler uses to resolve include directives in the source files. The `PUBLIC` keyword specifies that any targets that link to this target should also have access to that include directory. Other options are `PRIVATE`, and `INTERFACE` (the directory is not added to the include path for the specified target but is added to anything that links to it).

We can chain targets as follows:

```C
add_library(another STATIC another.cpp another.h)
target_link_libraries(another PUBLIC one)
```

[target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html) specifies that the `another` library is dependent upon the `one` library. Again, the `PUBLIC` keyword means that this link dependency is also propagated to any other targets that link to `another`. 

[target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html) is probably the most useful and confusing command in CMake. It can take more than the arguments we gave above - others include linker and compiler flags.

## Variables And The Cache
### Local Variables
A local variable is set like:

```C
set(MY_VARIABLE "value")
```

The names of variables are usually all caps, and the value follows. Variables are accessed using `${}`, such as `${MY_VARIABLE}`. CMake has the concept of [scope](../../Python/Scope%20in%20Python/Scope%20Types.md) - you can access the value of variable provided it's in scope. You can set a variable in the scope immediately above your current one with `PARENT_SCOPE` at the end of the `set()` command.

Lists are simply a series of values when you set them:

```C
set(MY_LIST "one" "two")
```

which internally become `;` separated values. So 

```C
set(MY_LIST "one;two")
```

is an identical statement. The [list()](https://cmake.org/cmake/help/latest/command/list.html) command has utilities for working with lists, such as:

```C
list(APPEND MY_LIST value)
```

### Variable Cache
CMake maintains a [variable cache](https://cmake.org/cmake/help/book/mastering-cmake/chapter/CMake%20Cache.html) as a way to store variables in a way that they are preserved between successive runs. We can think of it as a configuration file. The first time CMake is run on a project, it produces a `CMakeCache.txt` file in the top directory of the build tree. 

The idea is that this stores configuration options and user input variables, so that you don't have to re-list your options every time you run CMake.

To be clear: if you want to set a variable from the command line, CMake puts this in the variable cache. The syntax for declaring a variable and setting it if it's not already set is:

```C
set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "Description")
```

In this example, `"VALUE"` is the default value of the variable if it's not already set in the cache. `CACHE STRING` indicates the variable is a cache variable, and its type is `STRING`. `Description` is exactly that - a brief description of the variable, which can be displayed in GUIs and the like.

This mechanism allows us to set variables on the command line and not have them overridden when the CMake file executes. 

### Properties
The other way CMake stores information is in [properties](https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html). These are attributes attached to targets, source files, tests, or directories. There are various types of property, such as:

1. Global Properties (affect the entire project)
2. Directory Properties (affect a specific directory)
3. Target Properties (affect individual targets)
4. Source Properties (affect specific source files)
5. Test Properties (affect test targets)

You can set a property using the [set_property()](https://cmake.org/cmake/help/latest/command/set_property.html) command:

```C
set_property(TARGET TargetName
             PROPERTY CXX_STANDARD 11)
```

Which sets the `CXX_STANDARD` property of `TargetName` to `11`, meaning it'll be compiled with C++11 features enabled. Examples of commonly used properties include:

1. **INCLUDE_DIRECTORIES**: The directories to be added to the include path for a target or a source file.
2. **COMPILE_DEFINITIONS**: Compiler definitions to be added for a target or a source file.
3. **LINK_LIBRARIES**: The libraries to be linked against a target.
4. **CXX_STANDARD**: The C++ standard to be used for compiling a target.

## Programming in CMake
### Control Flow
CMake has an `if` statement, although what it tests for is a little complex. Here's an example:

```C
if(variable)
    # If variable is `ON`, `YES`, `TRUE`, `Y`, or non zero number
else()
    # If variable is `0`, `OFF`, `NO`, `FALSE`, `N`, `IGNORE`, `NOTFOUND`, `""`, or ends in `-NOTFOUND`
endif()
# If variable does not expand to one of the above, CMake will expand it then try again
```

CMake `if` statements support a range of comparison operators, including:

- `EQUAL`: Equal to
- `LESS`: Less than
- `GREATER`: Greater than
- `STRLESS`: String less than
- `STRGREATER`: String greater than
- `STREQUAL`: String equal to

as well as logical operators to combine conditions:

- `AND`
- `OR`
- `NOT`

### Generator Expressions
Generator expressions are powerful, but somewhat odd and specialised. They provide a way to evaluate expressions and conditions at the time of generating the build system, not when CMake is processing your `CMakeLists.txt` files. 

The basic syntax looks like:

```CMake
$<CONDITION:value-if-true,value-if-false>
```

They are enclosed within `$<>`. `CONDITION` is evaluated during the generation phase. Here's an example of how generator expressions can be used in practice:

```C
target_compile_definitions(mytarget PRIVATE
    $<$<CONFIG:Debug>:DEBUG_MODE>
)
```

In this example, the `DEBUG_MODE` definition is added to the `mytarget` target only if CMake is generating for a Debug build configuration. 

### Macros and Functions
You can define your own CMake `function` or `macro` easily. The only difference between them is scope; macros don't have their own scope, and instead use the scope they're called in.

An example of a function called `SIMPLE` is:

```C
function(SIMPLE REQUIRED_ARG)
    message(STATUS "Simple arguments: ${REQUIRED_ARG}, then ${ARGN}")
    set(${REQUIRED_ARG} "From SIMPLE" PARENT_SCOPE)
endfunction()
```


It requires at least one argument, called `REQUIRWED_ARG`. `ARGN` is a special variable used within functions/macros to refer to all arguments passed after the named arguments. This function is called like:

```C
simple(This Foo Bar)
message("Output: ${This}")
```

An equivalent macro would be called with:

```C
macro(SIMPLE REQUIRED_ARG)
    message(STATUS "Simple arguments: ${REQUIRED_ARG}, then ${ARGN}")
    set(${REQUIRED_ARG} "From SIMPLE" PARENT_SCOPE)
endmacro()
```

### Arguments
We can parse function and macro arguments using the [cmake_parse_arguments](https://cmake.org/cmake/help/latest/command/cmake_parse_arguments.html) function. This is particularly useful when a large number of arguments are passed. Here's an example:

```C
function(COMPLEX)
    cmake_parse_arguments(
        COMPLEX_PREFIX
        "SINGLE;ANOTHER"
        "ONE_VALUE;ALSO_ONE_VALUE"
        "MULTI_VALUES"
        ${ARGN}
    )
endfunction()

complex(SINGLE ONE_VALUE value MULTI_VALUES some other values)
```

Inside the function after this call, you'll find:

```C
COMPLEX_PREFIX_SINGLE = TRUE
COMPLEX_PREFIX_ANOTHER = FALSE
COMPLEX_PREFIX_ONE_VALUE = "value"
COMPLEX_PREFIX_ALSO_ONE_VALUE = <UNDEFINED>
COMPLEX_PREFIX_MULTI_VALUES = "some;other;values"
```

## Miscellaneous Commands
This section contains an explanation of a few common commands that I've seen (in Aladdin).

### option()
The [option()](https://cmake.org/cmake/help/latest/command/option.html) function is used to declare a Boolean cache entry. It's a convenient way to allow user to turn on or off certain features or build options in the CMake configuration[^fn1]. Its syntax is as follows:

```C
option(VARIABLE_NAME "Option description" DEFAULT_VALUE)
```


[^fn1]: The term "CMake configuration" refers to the process and set of `CMakeLists.txt` files that are used to configure a project's build. 

### add_subdirectory()
The [add_subdirectory()](https://cmake.org/cmake/help/latest/command/add_subdirectory.html) command adds a subdirectory to the build. It introduces another directory where CMake looks for additional `CMakeLists.txt` files to process. This is common in larger projects to manage the build process of different components, each residing in its own directory.

The basic syntax is:

```C
add_subdirectory(source_dir)
```

To be clear: when we call this function, CMake will both look for **and process** the `source_dir` CMakeLists.txt. We can then use its output (e.g. library files) at later points in our configuration process.
