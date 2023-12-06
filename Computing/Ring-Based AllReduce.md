These are just a few notes based on a [Medium article](https://towardsdatascience.com/visual-intuition-on-ring-allreduce-for-distributed-deep-learning-d1f34b4911da) on Ring-Based AllReduce. The idea is very simple, but nice to know about!

## Naive Approach
Clearly, an AllReduce requires every process to share data with every other process. Let's consider an example with 4 workers:

![](_attachments/Screenshot%202022-08-28%20at%2022.30.38.png)

The naive approach would be for each worker to communicate with every other worker, and then apply the reduction operation individually:

![](_attachments/Screenshot%202022-08-28%20at%2022.31.46.png)

It's pretty obvious that there are other approaches that require less communication.

## Using a Driver Process
Another approach would be to select one process to be the *driver*. The idea being that all processes send data to the driver; the driver can then apply the AllReduce, and redistribute the result.

![](_attachments/Screenshot%202022-08-28%20at%2022.33.35.png)

The problem with this approach is that it does not scale well; the driver process becomes a bottleneck since the required communication and computation (required to do the reduction operation) increases proportionally to the number of processes.

## Ring All-Reduce
The ring-based AllReduce has two phases:

1. Share-reduce phase;
2. Share-only phase

In the **share-reduce** phase, each process $p$ sends data to the next process. This connection resembles a ring. Additionally, the array of data of length $n$ is divided into $p$ chunks. Each of these chunks will be indexed by $i$ going forward.

![](_attachments/Screenshot%202022-08-28%20at%2022.45.06.png)

Specifically, we can write that processes $p$ sends data to the process $(p+1) \% p$.

The first share-reduce step has process A send $a_0$ to process B, process B send $b_1$ to process C, etc:

![](_attachments/Screenshot%202022-08-28%20at%2022.46.17.png)

When each process receives data from the previous process, it then applies the reduction operator and proceeds to send it to the next process in the ring. For instance, if our reduction operator is a sum:

![](_attachments/Screenshot%202022-08-28%20at%2022.48.08.png)

In order to do this, our reduction operation must be associative and commutative. Otherwise, the sum of intermediate results would not be equivalent to the full result.
The share-reduce phase ends when each process holds the complete reduction of chunk $i$. At this point, each process posseses a subset of the full result:

![](_attachments/Screenshot%202022-08-28%20at%2022.49.44.png)

The second step is the **share-only** step. Again, data is shared in a ring-like fashion. But there is no need to apply the reduction operation. This consolidates the result of each chunk in every process.