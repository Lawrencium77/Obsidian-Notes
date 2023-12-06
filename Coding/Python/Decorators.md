Notes made using [this](https://www.datacamp.com/community/tutorials/decorators-python) web page, and a [Corey Schafer video](https://www.youtube.com/watch?v=FsAPt_9Bf3U).

```toc
```

## Key Idea
Decorators allow a user to add new functionality to an existing object without changing its structure. They're usually called before the definition of a function you wish to decorate.

## Functions as First Class Citizens
In Python, functions are **first-class citizens**. This means they support all operations generally available to other objects. 
Examples include being passed as an argument, returned from a function, and assigned to a variable. 

The following give some examples.

### Assigning Functions to Variables
It is possible to assign a function to a *variable*, and use this variable to call the function. Note that we are assigned the function itself - *not* the result of the function - to the variable.

```python
def plus_one(num):
	return num + 1
	
add_one = plus_one
add_one(5)

>>> 6
```
### Defining functions inside other functions
Functions can also be defined *inside* another function:
```python
def plus_one(num):
	def add_one(num):
		return num + 1
	
	result = add_one(num)
	return result

plus_one(4)

>>> 5
```
### Passing Functions as Arguments to Other Functions
Functions can also be passed as parameters to other functions:
```python
def plus_one(num):
	return num + 1

def function_call(func):
	num_to_add = 5
	return func(num_to_add)

function_call(plus_one)

>>> 6
```
### Functions Returning Other Functions
A function can also generate other functions:
```python
def hello_function(): 
	def say_hi(): 
		return "Hi" 
	return say_hi 

hello = hello_function() 
hello()

>>> 'Hi'
```
### Nested Functions have access to the Enclosing Function's Variable Scope
Python allows a nested function to access the outer [scope](Scope%20in%20Python/Scope%20Types.md) of the enclosing function. This is an important concept in decorators, and is an example of [Closure](Scope%20in%20Python/Closure.md).
```python
def logger(msg):
	def log_message():
		print('Log:', msg)
	return log_message

log_hi = logger('Hi!')
log_hi()

>>> Log: Hi!
```
It is worth taking a moment to process this: the `log_hi()` function that we returned still has access to the `msg` variable, even after the outer function `logger` has finished executing.

Another way of thinking about this is that a **closure "closes over" the free variables from their environment.**
## Creating Decorators
Decorators take a function as an argument, wrap it to give some extra functionality, and return this new function. They do so using an enclosed function:
```python
def uppercase_decorator(function):
	def wrapper(): 
		func = function() 
		make_uppercase = func.upper() 
		return make_uppercase 
	return wrapper
```
We could then call this decorator as follows:
```python
def say_hi(): 
	return 'hello there' 

decorate = uppercase_decorator(say_hi) 
decorate()

>>> 'HELLO THERE'
```
This pattern ` decorated_function = decorator(function)` is typical. So much so, that Python provides some simpler syntax to apply these decorators, using the `@` symbol.
```python
@uppercase_decorator 
def say_hi(): 
	return 'hello there'
```
### Chaining Decorators
It is also possible to call multiple decorators:

```python

def star(func):
    def inner(*args, **kwargs):
        print("*" * 30)
        func(*args, **kwargs)
        print("*" * 30)
    return inner


def percent(func):
    def inner(*args, **kwargs):
        print("%" * 30)
        func(*args, **kwargs)
        print("%" * 30)
    return inner


@star
@percent
def printer(msg):
    print(msg)
```

The key point is that the above syntax of 
```python
@star
@percent
def printer(msg):
    print(msg)
```
is equivalent to 
```python
def printer(msg):
    print(msg)
printer = star(percent(printer))
```