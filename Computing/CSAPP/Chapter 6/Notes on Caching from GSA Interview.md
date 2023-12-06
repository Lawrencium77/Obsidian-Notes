## High-Frequency Trading and Compute Location

- **Computational Execution Point**: For operations like high-frequency trading where every nanosecond counts, computations should be carried out as close to the data source as possible. Ideally, this means performing computations on the **network adapter** itself rather than on the CPU to minimize latency. Although this isn't standard practice for all systems. 

## Cache Structure and Size

- **Uniform Cache Block Size**: Cache blocks across different levels of cache (L1, L2, L3) are standardised (often) at 64 bytes. This uniformity allows for predictability in cache behavior and simplifies the design of the caching system.

## Performance Comparison in Summing Arrays

- **C++ vs. Python Execution**: When summing integers in an array:
    - A C++ program storing `std::vector<int64>` is quick because the integers are stored contiguously in memory, allowing for efficient cache usage.
    - A Python program summing a list of `int64` values performs many orders of magnitude worse due to the nature of Python lists being arrays of pointers, leading to additional dereference steps and poor cache utilisation.

## Memory Access and Read Time Dominance

- **Initial Read Time**: The initial read time from RAM to L1 cache is the dominant factor since this is typically the slowest step in the memory hierarchy.
    
- **Acceleration Techniques**:
    
    - **L2 Prefetch Instructions**: These instructions can load data into L2 cache in parallel with ongoing computations to reduce wait times for data. However, this relies on having a good compiler.
    - **AI Chip for Access Pattern Prediction**: Instead, a dedicated AI chip can predict access patterns and perform prefetching accordingly. By handling prefetch logic off the primary CPU, we reduce L2 load time and improve overall read time. This is pretty cutting-edge and not widely adopted (03/11/23).

## Access Times and Cache Speeds

- **L1 Access Time**:
    
    - The access time for L1 cache is closely tied to the processor's clock speed. For a 3 GHz processor (0.3ns per clock cycle), L1 access times need to be in the ballpark of this clock period to match CPU speed and prevent stalling.
- **Cache Sizes**:
    
    - **L1 Cache**: Approximately 512 KB in size, designed for optimal speed to keep up with CPU execution.
    - **L2 Cache**: Roughly 10 MB, the access speed is typically only a few times slower than L1 cache. This can be reasoned about using typical cache block size - the L2 read speed needs to match the time it takes to process one block's worth of information.
- **RAM**: Typical sizes for RAM in systems handling large datasets, like those used in high-frequency trading, are around 500 GB, allowing for extensive data storage. A more typical value would be O(10 GB), although at Speechmatics our more advanced machines have O(1 TB).
    