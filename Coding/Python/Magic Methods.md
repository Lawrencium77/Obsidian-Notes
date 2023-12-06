
Also called **Dunder** methods.

They are designed such that they're not invoked directly by the user (although this is possible). Instead, the invocation happens internally from the class on a certain action.

E.g. when adding 2 numbers using the + operator, the `__add__()` method will be called internally.

Built-in classes in Python define many magic methods. The `dir()` function allows us to see all magic methods inherited by a class.

Magic methods are most often used to define overloaded behaviours of predefined operators in Python. An **overloaded** operator is one that has different definitions depending upon the operands used. For instance, the + operator is overloaded: for `int` objects, it defines addition but for `str` objects, it defines concatenation.

## Examples of Magic Methods
### `__new__()`
In Python, the `__new__()` method is implicitly called before the `__init__()` method when an instance of a class is created. The `__new__()` method returns a new object, which is then initialised by `__init__()`.
For example:

```py
class Employee:
    def __new__(cls):
        print ("__new__ magic method is called")
        inst = object.__new__(cls)
                return inst
    def __init__(self):
        print ("__init__ magic method is called")
        self.name='Satya'
		
		
>>> emp = Employee()  
__new__ magic method is called  
__init__ magic method is called
```

### `__str__()`
`__str__()` is used to return a printable string representation of any class.
When the `str()` built-in function is invoked, it called the  `__str__()` method.
As an example, we can overwrite the `__str__()` method in a class to return a different string representation of an object:

```py
class Employee:
    def __init__(self):
        self.name='Swati'
        self.salary=10000
    def __str__(self):
        return 'name='+self.name+' salary=$'+str(self.salary)
		
>>> e1=Employee()  
>>> print(e1)   
name=Swati salary=$10000
```

### `__add__()`
As explained above, this is used to define the behaviour of the overloaded + operator.