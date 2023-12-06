The [coroutine](https://en.wikipedia.org/wiki/Coroutine) is a general programming concept. 

Whilst there is no single precise definition of a coroutine, they have been described as **"functions whose execution you can pause"**. Two widely-acknowledged characteristics of a coroutine are:

1. The data local to a coroutine persist between successive calls.
2. The execution of a coroutine is suspended as control leaves it, only to carry on where it left off when control re-enters it at some later stage.

This is in contrast to a [subroutine](https://en.wikipedia.org/wiki/Function_(computer_programming)), which is just another term for a function[^fn1]. Subroutines don't remember state between invocations, and can't be paused. Instead, they simply run from beginning to end. When exited, any local state is lost.

Here's an example of simple coroutine in Python:

```python
def simple_coroutine():
    while True:
        value = (yield)  # Receive a value
        print('Received:', value)

c = simple_coroutine()
next(c)  # "prime" the coroutine
c.send('Hello')
```



### Difference with Generators
Although they look like functions, [generators](Iterators%20and%20Generators.md) are essentially just iterators. The distinction between generators and coroutines is that:

* The primary purpose of generators is to *produce* data that is consumed by something else.
* Coroutines, on the other hand can use the `yield` statement to produce data, but can also use `yield` as an expression in order to *consume* data.

So a generator is a specific type of coroutine - one that doesn't receive input. 

In Python, the distinction between generators and coroutines is a bit blurred since generators are sometimes said to have the ability to receive data. But the primary purpose and typical use cases are distinct. As GPT-4 puts it:

> [!QUOTE]
> In the end, the terms "generator" and "coroutine" are just labels that we use to talk about different patterns of usage for Python's `yield`.


> [!NOTE]
> I think it's worth pointing out that we are discussing two different types of coroutine here. The first is the general sense, i.e. a function who's execution you can pause. In this case, they're constrasted with subroutines.
> The second is the more narrow, Python sense, where we mean a function that can be paused, which is used only to consume data.



[^fn1]: Just a totally normal, ordinary function that you're used to using.