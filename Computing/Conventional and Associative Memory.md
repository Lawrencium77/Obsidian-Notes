I made some very brief notes on this topic. They're based on [Wikipedia](https://en.wikipedia.org/wiki/Content-addressable_memory), and CSAPP page 661.

#### Conventional Memory
* Is an array of values that takes an address as input, and return the value stored at that address.
* Also called **address-based memory**.
* RAM is an example.

#### Associative Memory
* Is an array of (key, value) pairs.
* Takes a key as input, and returns a the value associated with the matching key.
* Also called **Content-Addressable Memory**.
* **Hash maps are not an example of associative memory**. The hashing stage results in a bucket index, which is used to address conventional memory. 
