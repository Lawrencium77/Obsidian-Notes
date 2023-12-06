The difference between a mutex and a semaphore can be a classic interview question. These notes are based on [this](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/JQ5A5OLDZr9) Educative page, which I have improved upon.

## Mutex
[Mutex](https://en.wikipedia.org/wiki/Mutual_exclusion) is short for "mutual exclusion". A mutex is used to guard against shared data; it allows only a single thread to access a resource or [critical section](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/gkX0rjq1V6Y).

Once a [thread](The%20Operating%20System%20Manages%20the%20Hardware#Threads) acquires a mutex, all other threads attempting to acquire the same mutex are blocked until it's released by the first thread. 

In other words, they force threads to serialize their access to critical sections and shared data.

The following diagram illustrates the idea:

![](_attachments/Screenshot%202023-07-02%20at%2015.29.01.png)

## Semaphore
A [semaphore](https://en.wikipedia.org/wiki/Semaphore_(programming)) is used for limiting access to some resources. Think of a semaphore as having a limited number of "permits" to give out (aka the "count"); if a semaphore has given out all its permits, then any new thread requesting a permit will be blocked until an earlier thread with a permit returns it to the semaphore.

Here is a depiction of a semaphore with two permits:

![](_attachments/Screenshot%202023-07-02%20at%2015.34.07.png)

A semaphore with one permit is called a [binary semaphore](https://www.geeksforgeeks.org/binary-semaphore-in-operating-system/). This is often thought of as equivalent to a mutex, but the two are [slightly different](#Mutex%20vs%20Semaphore).

## Mutex vs Semaphore
One key difference between a mutex and semaphore is the concept of **ownership**: a mutex is owned by the thread that acquired it. The same thread must acquire and release a mutex. For semaphores, there is no concept of ownership; different threads can call acquire and release on the semaphore.

Another difference is that semaphores can be used for **signalling** among threads. This can be be understood by considering the [producer/consumer problem](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem) - a classic example in concurrent programming. 

The idea is that we've have two types of threads: producers, which create data, and consumers, which "consume" that data. The challenge is to ensure producers don't produce data faster than consumers use it, and vice versa. A semaphore can solve this problem: ^990b6a

1. Initialise the semaphore with `count = 0`
2. When a producer thread creates a piece of data, increment the semaphore count.
3. If a consumer thread is ready to consume data, it tries to decrement the semaphore. This works if `count > 0`.
4. Once the consumer thread has used the data, it repeats this process again, trying to decrement the semaphore.

The semaphore thus serves as a kind of signalling mechanism between producer and consumer threads. **The key reason that it can be used for signalling - whereas mutexes cannot - is that they remember how many times they have been acquired/released via their counter value.**

## Summary
* A mutex is used to serialize access to critical sections. 
* A semaphore can potentially be used as a mutex, but can also be used for signalling between threads.
* A mutex is **owned** by a thread, whereas semaphores don't support the concept of ownership.