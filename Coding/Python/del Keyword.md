A couple of notes on the `del` keyword in Python
```toc
```
### Everything is an Object

^6c564d

In Python, everything is an object. 
In other words, everything is an instance of a class.

For instance, a string is an attribute of the `str` class. A function is an attribute of the `function` class. And so on.

> [!INFO]
> Every object in Python has a unique id associated with it. The id is the object's memory address. To obtain the id for a specific object, we can use the `id()` function:
> ```python
> my_id = id(object)

### `del` Keyword
The `del` keyword in Python is used to delete an object. The object is removed from memory and the reference to the object is deleted.

It can be used to delete an [iterable](Iterators%20and%20Generators.md), or even **part** of an iterable. For instance:

```python
# Deleting a variable
x = 10
del x
print(x)  # This will raise an error as the variable x no longer exists

# Deleting a list item
fruits = ['apple', 'banana', 'cherry']
del fruits[1]
print(fruits)  # Output: ['apple', 'cherry']

# Deleting an element in a dictionary
# The key-value pair is removed
person = {'name': 'John', 'age': 30, 'city': 'New York'}
del person['age']
print(person)  # Output: {'name': 'John', 'city': 'New York'}
```

It's important to understand that the `del` keyword removes an object's reference, not the object itself. The object remains in memory until its reference count drops to zero, at which point it becomes eligible for [Garbage Collection](Garbage%20Collection.md).