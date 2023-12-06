The Linux **signal** is a higher-level software form of exceptional control flow. It allows processes and the kernel to interrupt other processes.

A **signal** is a small message that notifies that an event of some type has occurred. Figure 8.26 shows the 30[^fn1] different types of signals that are supported on Linux systems:

![](_attachments/Screenshot%202023-07-16%20at%2013.56.43.png)

Each signal type corresponds to some kind of event. Low-level hardware exceptions are processed by the kernel's exception handlers and would not normally be visible to user processes. Signals provide a mechanism for exposing the occurrence of such exceptions to user processes.

For instance, if a process attempts to divide by zero, then the kernel sends it a `SIGFPE` signal.

### Signal Terminology
The transfer of a signal between processes occurs in two steps:

1. **Sending a signal**: The kernel updates some state in the context of the destination process[^fn2].
2. **Receiving a signal**: a destination process is forced by the kernel to react in some way to the delivery of a signal. The process can either ignore it, terminate, or **catch** the signal by executing a user-level function called a **signal handler**:

![](_attachments/Screenshot%202023-07-16%20at%2014.09.00.png)

A signal that has been sent but not received is called a **pending** signal. A process can selectively **block** the receipt of certain signals. When a signal is blocked, it can be delivered, but the resulting pending signal will not be received by the destination process.

## Sending Signals
Unix systems provide a number of mechanisms for sending signals to processes. All of the mechanisms rely on the notion of a **process group**.

#### Process Groups
Every process belongs to exactly one **process group**. It's identified by a positive integer, the **process group ID (PGID)**. By default, a child process belongs to the same process group as its parent. A process can change the process group of itself or another process.

#### Kill
The `/bin/kill` program sends an arbitrary signal to another process. For example:

```
linux> /bin/kill -9 15213
```

sends signal 9 (`SIGKILL`) to process 15213. A negative PID causes the signal to be sent to every process in the process group PGID. In other words:

```
linux> /bin/kill -9 -15213
```

sends `SIGKILL` to every process in process group 15213.

#### Sendings Signals from the Keyboard
Inside a Unix shell, a **process group** is usually called a **job**. A separate job is created as a result of evaluating a single command line prompt. 

At any point in time, there is at most one foreground job and zero or more background jobs. For instance, typing:

```
linux> ls | sort
```

creates a foreground job consisting of two processes connected by a pipe. 

Figure 8.28 shows a shell with one foreground job and two background jobs:

![](_attachments/Screenshot%202023-07-16%20at%2014.24.28.png)

The parent process in the foreground job has a PID of 20 and a process group ID of 20. The parent process has created two children, each of which are also members of process group 20.

Typing `Ctrl + C` causes the kernel to send a `SIGINT` signal to every process in the foreground process group. In the default case, the result is to terminate every foreground job. Similarly, typing `Ctrl + Z` causes the kernel to send `SIGTSTP` to every process in the foreground process group. This suspends the foreground job.

### Receiving Signals
When the kernel switches a process $p$ from kernel mode to user mode, it checks the set of unblocked pending signals for $p$. If this set is empty, the kernel passes control to the next instruction, $I_\textrm{next}$. If the set is nonempty, the kernel chooses some signal $k$ in the set and forces $p$ to **receive** signal $k$.
Each signal type has a predefined *default action*, which is one of:

* Process terminates
* Process terminates and dumps core
* Process stops until restarted by a `SIGCONT` signal
* Process ignores the signal

A process can modify the default action associated with a signal using the `signal` function. The only excepts are `SIGSTOP` and `SIGKILL`, whose default actions cannot be changed.

The `signal` function can change the action associated with a signal `signum` in one of three ways:

* If `handler` is `SIG_IGN`, then signals of type `signum` are ignored.
* If `handler` is `SIG_DFL`, then the action for signals of type `signum` reverts to the default action.
* Otherwise, `handler` is the address of a user-defined function, called a **signal handler**. Changing the default action by passing the address of the handler is called **installing the handler**. The invocation of the handler is called **catching the signal**. The execution of the handler is referred to as **handling the signal**.

Actually implementing this in C is very simple:

```C
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void sigint_handler(int sig_num)
{
    /* Reset handler to catch SIGINT next time. */
    signal(SIGINT, sigint_handler);
    printf("\n Caught SIGINT signal. Press Ctrl+D to exit. \n");
    fflush(stdout);
}

int main ()
{
    /* Register signal and signal handler */
    signal(SIGINT, sigint_handler);
    
    while(1)
    {
        printf("Running indefinitely. Press Ctrl+C to generate SIGINT signal.\n");
        sleep(1);
    }
    
    return 0;
}
```

The `signal()` function installs `sigint_handler()` as the handler for the `SIGINT` signal. Pretty simple.

### Blocking and Unblocking Signals
I skipped this part.

### Writing Signal Handlers
I skipped this part too. Although it does look very interesting.

### Synchronizing Flows To Avoid Concurrency Bugs
This touches on the topic of [concurrency](../../Concurrency/Educative%20Course/Concurrency%20vs%20Parallelism.md), which is the topic of Chapter 12. I left it for now.

### Explicitly Waiting for Signals
Sometimes, a program needs to explicitly wait for a certain signal handler to run. For example, when a Linux shell creates a foreground job, it must wait for the job to terminate and be reaped by the `SIGCHLD` handler before accepting the next user command.

This part of the book discusses how to implement this. Again, I left it for now.




[^fn1]: The exact number that are actually supported varies a bit between Linux versions.
[^fn2]: The implementation details here aren't too important. 