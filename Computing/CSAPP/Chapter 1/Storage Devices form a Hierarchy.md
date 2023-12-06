The notion of inserting a smaller, faster storage device (e.g. cache) between the processor and a larger, slower device (e.g. main memory) turns out to be a general idea. In fact, storage devices in every computer system are organized as **memory hierarchy** similar to Figure 1.9. As we move from the top to bottom, the devices become slower, larger and cheaper. The register file occupies the top level, which is known as L0. The three levels of cache occupy L1-3. Main memory occupies L4, and so on.

The main idea of the memory hierarchy is that **storage at one level serves as cache for storage at the next lower level**. Thus, the register file is a cache for the L1 cache. Caches L1 and L2 are caches for L2 and L3, respectively. And so on.

![](_attachments/Screenshot%202022-05-13%20at%2012.34.39.png)
