I made these notes whilst reading & making [notes](The%20Linux%20Kernel/0%20-Intro.md) about the Linux kernel.

An executable file can have many formats, or even be a script file. Script files have to be recognised and the appropriate interpreter run to handle them; for instance, `/bin/sh` interprets shell scripts. Linux uses the standard Unix convention of having the first line of a script file contain the name of the interpreter. So a typical script file would start:

```shell
#! /bin/bash 
```

The **shebang** is a special line at the top of a shell script that specifies the interpreter that should be used to run the script.[^fn1] It consists of a `#` followed by `!`.
In addition to this, the shebang can also be used to specify certain command-line options. For instance, adding the `-e` option like so:
```shell
#! /bin/bash -e
```
 
would cause the interpreter to stop executing if any command fails.

At Speechmatics, we use zsh as our command line shell, but bash as our shell script interpreter. We use zsh because we can get **oh-my-zsh** completions, and bash in shell scripts because it is more ubiquitous.

[^fn1]: The interpreter is just another shell process. See [Running Shell Scripts](Running%20Shell%20Scripts.md) for more info.