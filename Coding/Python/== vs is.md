Some very brief notes on singleton objects and the difference between the `==` and `is` operators.

### Singleton Objects
In Python, a singleton object is an object that's instantiated only once. All references to the object refer to the same instance in memory.

### == vs is
In Python, `is` checks for object identity comparison. It checks that two [objects](del%20Keyword#^6c564d) have the same memory address.

On the other hand, `==` checks for the equality of the values of two objects, without necessarily checking that they're the same object in memory.

### Checking for None
When checking for `None` in Python, it is recommended to use `is None` instead of `== None` because `None` is a singleton object. Using `==` to check for `None` would work in most - but not all - cases. For instance, if we define a class that overrides the `==` operator, the behaviour of `==` may not be the same as `is`. Therefore, it's generally safer to use `is None`.





