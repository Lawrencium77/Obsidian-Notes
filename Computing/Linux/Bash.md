While digging into how dotfiles work, I decided there were a few Bash-related concepts that I didn't properly understand. I therefore decided to read the Educative course on [Bash for Programmers](https://www.educative.io/courses/bash-for-programmers/xVlj21oQQ2P) and pick out the useful parts.
These notes are a summary of what I learned but skip the parts I was familiar with.
I also augmented a few of the points with reading from other sources.

```toc
```

# Introduction
What is a shell? **A shell is a computer program that exposes an operating system's services to a user.** In general OS shells either use a command-line interface or a graphical user interface. It is named a shell because it is the outermost layer around the OS.

Let us be clear: a shell is *not an operating system*. It is a way to interface with the OS and run commands.

A `Unix` shell is both a command interpreter and a programming language. As a command interpreter, it provides the UI to the substantial set of GNU utilities. The programming languages features allow the utilities to be combined.

[This book](https://tldp.org/LDP/tlk/kernel/processes.html) on the Linux kernel gives quite an instructive summary of what a shell is:

> [!INFO]
> In Linux programs and commands are normally executed by a command interpreter. A command interpreter is a user process like any other process and is called a shell.
> 
> For each command entered, the shell searches the directories in the process's _search path_, held in the PATH environment variable, for an executable image with a matching name. If the file is found it is loaded and executed. The shell clones itself using the _fork_ mechanism and then the new child process replaces the binary image that it was executing, the shell, with the contents of the executable image file just found. Normally the shell waits for the command to complete, or rather for the child process to exit. 
> 
> You can cause the shell to run again by pushing the child process to the background by typing control-Z, which causes a SIGSTOP signal to be sent to the child process, stopping it. You then use the shell command bg to push it into a background, the shell sends it a SIGCONT signal to restart it, where it will stay until either it ends or it needs to do terminal input or output.
> 
> An executable file can have many formats or even be a script file. Script files have to be recognized and the appropriate interpreter run to handle them; for example /bin/sh interprets shell scripts. Executable object files contain executable code and data together with enough information to allow the operating system to load them into memory and execute them.

#### What is Shell Scripting?
We can put a set of commands in a plain text file with the extension of `.sh`. These files are shell scripts which can be executed in `Unix`.

#### Types of Shell
In `Unix`, there are two major types of shell:

![|500](_attachments/Screenshot%202022-12-04%20at%2010.35.34.png)

#### Relationship to Linux
Linux is a `Unix` clone. Its primary developer with Linus Torvalds. Strictly speaking, it is a kernel and not an entire OS.

#### Command Line and Graphical Shells
Broadly speaking, we can split shells into Command line and Graphical shells:

![|500](_attachments/Screenshot%202022-12-04%20at%2010.25.07.png)

# Processes
A process refers to a program in execution; it's a running instance of a program. 
The best explanation I've found is from [these notes](https://tldp.org/LDP/tlk/tlk-toc.html) on the Linux kernel:

> [!INFO]
> Processes carry out tasks within the operating system. A *program* is a set of machine code instructions and data stored in an executable image on disk and is, as such, a passive entity; a *process* can be thought of as a computer program in action.
> 
> It is a dynamic entity, constantly changing as the machine code instructions are executed by the processor. As well as the program's instructions and data, the process also includes the program counter and all of the CPU's registers as well as the process stacks containing temporary data such as routine parameters, return addresses and saved variables. 
> 
> The current executing program, or process, includes all of the current activity in the microprocessor. Linux is a multiprocessing operating system. Processes are separate tasks each with their own rights and responsibilities. If one process crashes it will not cause another process in the system to crash. Each individual process runs in its own virtual address space and is not capable of interacting with another process except through secure, kernel managed mechanisms.

Every process in the system (except the initial process) has a parent process. New processes are not created; they are copied, or rather *cloned* from previous processes. Every process keeps pointers to its parent process, its siblings and its own child processes.

A **group of processes** running in series or parallel is considered a **job**.

# Variables and Environment
Depending on their lifetime, range, and uses, Bash variables can generally be divided into three categories:

1. Shell variables
2. Local variables
3. Environment variables

#### Shell Variables
Shell variables are a combination of local variables which are required by the Shell. Each Shell has its own Shell variables which are only accessible inside that Shell. 
An example is `PWD`: this is available only within the current instance of a shell. It is not exported across all instances of the shell.

#### Local Variables
Local variables are only accessible inside the current shell. They cannot be accessed by child processes which run under the current shell. All user-defined variables are local variables.

#### Environment Variables
Environment variables can be accessed anywhere in the Shell, and even by child processes called in a shell. In Bash, the `export` command is used to create environment variables.

E.g. `$PATH` is on of the most important environment variables. It specifies a list of directories where programs are stored. 

The `env` command in zsh prints all environment variables that are set in the current shell.

> [!INFO] 
> The PID of the init process is always 1 in Linux.

> [!QUESTION]
> I'm not exactly clear on the distinction between shell and environment variables in modern zsh. I don't know if child processes inherit shell variables or not.

