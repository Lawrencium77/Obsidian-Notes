```toc
```
## Overview

A **hash map**, also called a **hash table**, is a data structure that implements an **associative array**[^fn1]. It uses a **hash function** to compute an **index**, also called a **hash code**, into an array of **buckets** from which the desired value can be found. During lookup, the key is hashed and the resulting hash indicates where the corresponding value is stored.

Here's a diagram of a small phone book, implemented as a hash map:

![](_attachments/Screenshot%202022-08-20%20at%2014.56.36.png)

Ideally, the hash function will assign each key to a unique bucket. However, most hash table designs use an imperfect hash function, which might cause **hash collisions** where the hash function generates the same index for more than one key. Such collisions are typically accommodated in some way.

In a well-dimensioned hash map, the average time complexity for lookup, insertion, and deletion is $O(1)$.

At the core of hashing is a **space-time tradeoff**. If memory is infinite, the *entire key* can be used directly as an index to locate its value with a single memory address. On the other hand, if infinite time is available, values can be stored without regard for their keys, and some search algorithm can be used to retrieve the element.

Hash tables are commonly used to implement **sets**, by omitting the stored value for each key and merely tracking whether the key is present.

### Load Factor
The **load factor** $\alpha$ is a critical statistic of a hash table, and is defined as follows:

$\textrm{load factor}(\alpha)=\frac{n}{k}$

where $n$ is the number of entries occupied in the hash table, and $k$ is the number of buckets.

The performance of hash tables deteriorates in relation to $\alpha$. A hash map is resized or **rehashed** if $\alpha$ approaches 1, or drops below $\alpha_{\textrm{max}}/4$. A value of $\alpha$ that typically works well is 0.75.

## Choosing a Hash Function
A perfect hash function would map all keys to a unique index. This can only be created if all keys are known ahead of time.

A uniform distribution of the hash values is a fundamental requirement of a hash function. A non-uniform distribution will make hash collisions more likely.

## Collision Resolution
A search algorithm that uses hashing consists of two parts:

1. Computing a hash function which transforms the search key into an array index. 
2. The second part is collision resolution.

There are multiple methods of collision resolution. The two most common are **separate chaining** and **open addressing**. 

### Separate Chaining
In separate chaining, the process involves building a **linked list**[^fn2] of key-value pairs. The collided items are chained together through a single linked list, which can then be traversed to access the item with the correct search key.

![](_attachments/Screenshot%202022-08-20%20at%2015.20.03.png)

### Open Addressing
Open addressing is similar to separate chaining in that it also involves traversing a sequence of items. However, the difference is the every entry is stored *in the bucket array itself*. The hash resolution is performed through  **probing**.

When a new entry has to be inserted, the buckets are examined, starting with the hashed-to slot and processing in some *probe sequence*, until an unoccupied slot is found. When searching for an entry, the buckets are scanned in the same sequence, until either the target record is found, or an unused array slot is found,  which indicates an unsuccessful search.

Well-known probe sequences include:

* **Linear probing**: the interval between probes is fixed.
* **Quadratic probing**: the interval between probes grows.
* **Double hasing**: the interval between probes is computed by a secondary has function.

Below is an example of open addressing with linear probing (interval $=1$):

![](_attachments/Screenshot%202022-08-20%20at%2015.25.29.png)






[^fn1]: An associative array is another term for a *dictionary*: it is a data type that stores a collection of (key, value) pairs, with the constraint of unique keys. There are two major approaches to implementing dictionaries: *hash tables* and *search trees*.
[^fn2]: A linked list is a linear collection of elements whose order isn't given by their physical placement in memory. Instead, each element points to the next.