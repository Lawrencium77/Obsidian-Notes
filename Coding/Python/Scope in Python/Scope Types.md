These notes cover types of variable scope in Python, along with the significance of the `nonlocal` and `global` keywords.

These concepts are also covered in [this part](Educative%20Course.md#^731284) of the notes I made about C.

```toc
```

## Variable Scope
A variable scope defines the region in which we can access a variable.
In Python we can declare variables with three different scopes:

* Local
* Global
* Nonlocal

There is one extra scope type:

* Built-in

## Local Variables
When we declare variables inside a function, they have **local** scope. We cannot access them outside the function. For instance:

```python
def greet():
	message = "hello" # Local variable
	print(f"Local, {message}")

greet()

# Try to access the message variable
print(message)
```

Output:

```
Local, hello
NameError: name 'message' is not defined
```

> [!INFO]
> Whenever you try to access a name that isn’t defined in any Python scope, you’ll get a `NameError`. 


## Global Variables
In Python, a variable defined outside of the function is a global variable.
It can be accessed anywhere within the program, including inside other functions.
For instance:

```python
message = 'Hello'  # declare global variable

def greet():
    print('Local', message)  # access global variable

greet()
print('Global', message)
```

Output:

```
Local Hello
Global Hello
```

## Global Keyword
In Python, we can *access* global variables from within a function. But we cannot *modify* them.
In order to do this, we need the `global` keyword. The key point is that:

> [!IMPORTANT]
> The `global` keyword allows us to create global variables from a non-global scope.

As an example, if we tried to run this code:

```python
c = 1 

def add():
    c += 2 
    print(c)

add()
```

we'd get an error:

```
UnboundLocalError: local variable 'c' referenced before assignment
```

But using the global keyword allows us to modify the global variable:

```python
c = 1 # global variable
def add():
    global c # declare global variable with same name as c
    c += 2 
    print(c)
    
add()

# Output: 3 
```

### Global Inside Nested Function
We can also use `global` inside a nested function:

```python
def outer_function():
    num = 20

    def inner_function():
        global num
        num = 25
    
    print("Before calling inner_function(): ", num)
    inner_function()
    print("After calling inner_function(): ", num)

outer_function()
print("Outside both function: ", num)
```

Output:

```
Before calling inner_function():  20
After calling inner_function():  20
Outside both function:  25
```

In this example, we declared a global variable `num` inside the nested function `inner_function()`. 

## Nonlocal Variables
Nonlocal variables are used in nested functions whose local scope is not defined. The outer function's variables are neither local nor global in the scope. For instance:

```python
def outer():
    message = 'nonlocal'
    def inner():
        message = 'local'
        print("inner:", message)

    inner()
    print("outer:", message)

outer()
```

Output:

```
inner: local
outer: nonlocal
```

Nonlocal scope is sometimes called **enclosing scope**.

### Nonlocal Keyword
Using the nonlocal keyword allows us to create nonlocal variables from a local scope:

```python
def outer():
    message = 'local'
    def inner():
	    nonlocal message
        message = 'nonlocal'
        print("inner:", message)

    inner()
    print("outer:", message)

outer()
```

Output:

```
inner: nonlocal
outer: local
```


## A Nice Example
I thought that this code summarises the relationship between local, nonlocal, and global variables quite well:

```python
x = 0
def outer():
    x = 1
    def inner():
        x = 2
        print("inner:", x)

    inner()
    print("outer:", x)

outer()
print("global:", x)
```

Output:

```
inner: 2
outer: 1
global: 0
```

By using the `global` and `nonlocal` keywords, we are able to modify the output of this code as we wish. Give it a go!

> [!INFO]
> The important thing to remember is that the **name** of the variable is irrelevant. It is the **scope** of the variable that determines where it can be accessed.

## Built-in Scope
Built-in scope is a special Python scope that's loaded whenever we run a Python program. It contains attributes that are built into Python. Names in this Python scope are available everywhere in your code.

They include names like `False`, `None`, `and`, `pass`, and so forth.

## Viewing local and global variables
The `globals()` function returns a dictionary the names & values of all the current global variables:

```python
score = 23
globals()['score'] = 10
print('The Score is:', score)
```

Likewise, the `locals()` function displays the current state of the local scope. It makes sense to call it from within a function:

```python
def test():
    Language = "Python Programming"
    print("Local variable: ", locals())
```


## Still to Cover:
* `dir()`, `vars()`, and namespaces
* [This website](https://realpython.com/python-scope-legb-rule/#using-the-legb-rule-for-python-scope) has a decent amount of detail. As does [this](https://www.knowledgehut.com/blog/data-science/python-scopes).