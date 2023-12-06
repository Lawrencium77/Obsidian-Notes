I made these notes by picking the important bits from [Wikipedia](https://en.wikipedia.org/wiki/Load_(computing)).

## What is Load?
In UNIX computing, the system **load** is a measure of the amount of computational work that a system performs. It is measured using the **load average**.

The load average is calculated as the average number of [processes](Linux/The%20Linux%20Kernel/4%20-%20Processes.md) that are in either a **runnable** or **uninterruptible sleep** state. Linux systems typically calculate the load average over three different periods: 1, 5, and 15 minutes.

This information can be obtained using the `uptime` command:

```
> uptime
12:01:03 up 357 days, 17:34,  2 users,  load average: 1.73, 1.45, 1.44
```

This load information can also be found in the `/proc/loadavg` file.

What qualifies as a "high" load is not well-defined. Generally speaking, if the load average is less than the number of CPU cores, the system is not overloaded.