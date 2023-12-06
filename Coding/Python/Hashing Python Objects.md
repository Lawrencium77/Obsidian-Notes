These notes are largely based on [this StackOverflow answer](https://stackoverflow.com/questions/14535730/what-does-hashable-mean-in-python).

Some Python objects can be **hashed**. For example:

```python
my_string = "12345"
print(hash(my_string))
print(my_string.__hash__()) 

> 6527852807866460971
> 6527852807866460971
```

Hashability makes an object usable as a [hash map](../../Computing/Hash%20Maps.md) key. It can therefore be used as a dictionary key, or as a member of a set.

All of Python's [immutable](../(Im)mutability.md) built-in objects are hashable, while mutable containers (such as lists or dictionaries) are not. 

Note that this **not** the same thing as `id()`, which essentially returns the address of an object in memory.

### Hashing Seeds
There is an important detail to what we've been discussing. 

The `hash()` function is not designed to return the same value across different Python processes/runs. Python adds a random seed to the hash function at the start of each process (for security reasons).

This means that if you run the code:

```python
str = '123'
hash(str)
```

in two different Python processes, you may get different hash values.

We can control this behaviour using the `PYTHONHASHSEED` environment variable. If you set `PYTHONHASHSEED` to `0` or any fixed integer, it turns off the randomisation, making the `hash()` function deterministic.

So we could do this with something like:

```bash
export PYTHONHASHSEED=0
python your_script.py
```


