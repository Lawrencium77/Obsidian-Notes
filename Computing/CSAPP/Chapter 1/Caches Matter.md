An important lesson from this example is that the system spends a lot of time moving information from one place to another. From a programmer's perspective, much of this copying is overhead that slows down the "real work" of the program. Thus, a major goal for system designers is to make these copy operations run as fast as possible.

Because of physical laws, larger storage devices are slower than smaller storage devices. And faster devices are more expensive to build. For example, the disk drive on a typical system might be 1,000 times larger than the main memory, but it might take the processor 10,000,000 times longer to read a word from disk than from main memory.
Similarly, a typical register file stores only a few hundred bytes of data, as opposed to billions of bytes in the main memory. However, the processor can read data from the register file almost 100 times faster than from memory.

Even more troublesome is that as semiconductor technology progresses, this *processor-memory* gap continues to increase. It is easier and cheaper to make processors run faster than to make main memory run faster.

To deal with the processor-memory gap, system designers include smaller, faster storage devices called **caches**. These serve as temporary staging areas for information that the processor is likely to need in the near future. Figure 1.8 shows the cache memories in a typical system.

![](_attachments/Screenshot%202022-05-13%20at%2012.22.07.png)

An **L1 cache** on the processor chip holds tens of thousands of bytes and can be accessed nearly as fast as the register file. A larger **L2 cache** with millions of bytes is connected to the processor by a special bus. It might take the processor 5-10 times longer to access the L2 cache than the L1 cache, but this is still 5-10 times faster than accessing main memory. 

The L1 and L2 caches are implemented with **SRAM**. Newer and more powerful systems even have three levels of cache: L1, L2 and L3. The idea behind caching is that a system can get the effect of both a very large memory and a very fast one by exploiting **locality**: the tendency for programs to access data and code in localized regions. By setting up caches to hold data that are likely to be accessed often, we can perform most memory operations using fast caches.

One of the most important lessons in this book is that application programmers who are aware of cache memories can exploit them to improve the performance of their programs by an order of magnitude.
