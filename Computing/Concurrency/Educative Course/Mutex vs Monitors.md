In short: a [monitor](https://en.wikipedia.org/wiki/Monitor_(synchronization)) is a mutex and then some. It is another type of synchronisation construct, such as [mutex or semaphore](Mutex%20vs%20Semaphore.md).

Monitors are generally language-level constructs, whereas mutexes and semophores are more low-level. 

## What Problem Do Monitors Solve?
Consider the [producer/consumer application](Mutex%20vs%20Semaphore.md#^990b6a). The consumer thread cannot consume until the producer has produced some data. 

We can solve this with a mutex. The logic goes as follows:

* When the producer threads runs, it must wait on a flag that indicates if the buffer has space.
* When the consumer thread runs, it must wait on a flag if the buffer has any data. 
* To accomplish this, we design both threads to repeatedly check for the predicate to be `True`. The corresponding pseudocode is:

```C
// Shared Resources
buffer = []
bufferSize = N
mutex = Mutex()

Producer() {
    while (true) {
        item = ProduceItem()
        while (true) {
            mutex.lock()
            if (buffer.size() < bufferSize) {
                buffer.add(item)
                mutex.unlock()
                break  // Exit the spinlock
            }
            mutex.unlock()
        }
    }
}

Consumer() {
    while (true) {
        while (true) {
            mutex.lock()
            if (buffer.size() > 0) {
                item = buffer.remove()
                mutex.unlock()
                break  // Exit the spinlock
            }
            mutex.unlock()
        }
        ConsumeItem(item)
    }
}

```

Consider the following scenario: the `Producer()` thread has produced an item, and would like to add it to the buffer, but the buffer is full and the `Consumer()` thread is busy consuming. The `Producer()` thread acquires the mutex, sees that the buffer is full, so then releases it.
A similar chain of events occurs if the `Consumer()` event is waiting on the buffer to be non-empty.

This code works, but is an example of "spin waiting" (aka "[busy waiting](https://en.wikipedia.org/wiki/Busy_waiting)" aka "spinning"). We **waste a lot of CPU cycles** simply releasing/acquiring mutexes, and evaluating predicates. So our program is less efficient.

## Condition Variables
How can we improve this? We want to test for a predicate with a having acquired a mutex so that other threads can't change the predicate when we test for it. But what if, when find the predicate to be `False`, we **wait** on a **condition variable** until the predicate changes? In other words, we put the thread in some kind of sleep state, where it waits until the predicate changes state. 

This is the solution to spin waiting.

Conceptually, each condition variable exposes two methods: `wait()` and `signal()`:

* The `wait()` method causes the associated mutex to be atomically released, and the thread is put on a **wait queue**. In general, there could be other threads in this queue.
* Eventually, predicate changes state and evaluates to `True`.
* The thread responsible for this change calls `signal()` on the condition variable.
* This method causes the waiting thread to "get ready" for execution. Note that it *does not* start executing immediately. It is just put into a **ready queue**.
* The running thread eventually releases the mutex, and the waiting thread can now start executing.

In pseudocode:

```C
//Shared Resources
buffer = []
bufferSize = N
mutex = Mutex()
notEmpty = Condition()
notFull = Condition()

Producer() {
    while (true) {
        item = ProduceItem()
        mutex.lock()
        while (buffer.size() == bufferSize)
            notFull.wait(mutex)
        buffer.add(item)
        notEmpty.signal()
        mutex.unlock()
    }
}

Consumer() {
    while (true) {
        mutex.lock()
        while (buffer.size() == 0)
            notEmpty.wait(mutex)
        item = buffer.remove()
        notFull.signal()
        mutex.unlock()
        ConsumeItem(item)
    }
}

```

> [!INFO]
> You may wonder why we can't now use an `if` statement instead of a `while` statement in the above pseudocode. To understand better, see the [next section](Mesa%20vs%20Hoare%20Monitors.md).

## So What is a Monitor?
A monitor is made up of a **mutex** and $\geq 1$ **condition variables**. 

Another way to think of them is as an entity having multiple queues (aka "sets") where threads can be placed. One is the **entry set**; the others are **wait sets** - we've one of these for each condition variable. 

When thread `A` **enters** a monitor, it is placed into the entry set. If no other thread **owns** the monitor, i.e. is actively executing within the monitor section (c.f. owning a mutex), then thread A **acquires** the monitor. While thread `A` owns the monitor, no other thread will be able to execute any of the critical sections protected by the monitor. New threads requesting ownership get placed into the entry set.

Thread `A` continues to execute within the monitor section until it **exits** the monitor, or calls `wait()` on a condition variable and is placed into its wait set. Suppose another thread `B` enters the monitor and is put in the entry set. It will successfully acquire the monitor and continue execution. If thread `B` calls `signal()`, it will exit the monitor. At this point, thread `A` can acquire the monitor again and execute, this time with the predicate presumably set to `True`.

**You can think of monitors as a mutex with some number of wait sets** (a mutex already has an entry set). 