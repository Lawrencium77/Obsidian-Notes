## Mesa Monitors

When looking at [Mutex vs Monitors](Mutex%20vs%20Monitors.md), we established that monitors require a `while` loop, e.g:

```C
while (buffer.size() == bufferSize) 
	notFull.wait(mutex)
```

Once the waiting threads has been readied, why does it need to check for the `predicate` being `false` again? Has the signalling thread not just set the condition to be `true`?

In **Mesa monitors**, the readied thread `A` competes with other threads to acquire the mutex once the signalling thread `B` **empties** the monitor. For this reason, we must use a `while` instead of `if` statement.

This is sometimes called **spurious wakeup**.

## Hoare Monitors
In contrast, **Hoare monitors** ensure that the signalling thread `B` **yields** the monitor to the readied thread `A`. Thread `A` then enters the monitor. This guarantees that the predicate will not have changed, meaning that an `if` clause would suffice.

> [!INFO]
> Mesa monitors are more efficient than Hoare monitors. Python uses them. 