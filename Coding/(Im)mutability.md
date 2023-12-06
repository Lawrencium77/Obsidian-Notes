An [immutable](https://stackoverflow.com/questions/3200211/what-does-immutable-mean#:~:text=Immutable%20means%20the%20value%20can,be%20modified%20as%20its%20immutable.) object is an object whose value cannot be changed after it has been created. 

For example, Python's basic data types like `int`, `float`, `str`, and `tuple` are all immutable. Once you create these objects, you can't modify them. If you try to modify them, Python actually creates a new object with the modified value and assigns it to the existing variable.

Here's an example:

```python
s = "hello"
id1 = id(s)

s += " world"
print(s)  # Outputs: hello world
id2 = id(s)

print(id1 == id2) # False
```


In contrast, some objects like `list`, `dict`, or instances of custom classes are mutable:

```python
l = [1, 2, 3]
print(id(l))  # This might output: 1396585864135

l.append(4)
print(l)  # Outputs: [1, 2, 3, 4]
print(id(l))  # Outputs same id as before: 1396585864135
```

### Why use an Immutable Object?
Using immutable objects can be very useful. Two reasons include:

* [Hashability](Python/Hashing%20Python%20Objects.md): Only immutable objects can be used as dictionary keys (or members of a `set`) in Python. This is because the hash of an object must remain constant while it is in use as a dictionary key to ensure correct behaviour. Since immutable objects can't change, their hash is guaranteed to stay the same.
* [Thread Safety](../Computing/Concurrency/Educative%20Course/Mutex%20vs%20Monitors.md): since immutable objects cannot be changed, they will always be thread safe.

There are many more!

### Why use a Mutable Object?
The main reason for this is simply that mutable objects allow for **in-place changes, without the need for creating a new object**. This is useful for large data structures, where copying an object would be expensive.


