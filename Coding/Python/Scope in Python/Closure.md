In general, a closure can be described as follows:

> [!QUOTE]
> The combination of a function combined (i.e. *enclosed*) with references to its surrounding state (the *lexical environment*).

Put simply: it's a function that's able to access other variables local to the [scope](Scope%20Types.md) it was created in.

In Python, a closure is created when a function is defined in another function, allowing the inner function to access variables in the outer one:

```python
def logger(msg):
	
	def log_message():
		print('Log:', msg)
		
	return log_message

log_hi = logger('Hi!')
log_hi()

>>> Log: Hi!
```
