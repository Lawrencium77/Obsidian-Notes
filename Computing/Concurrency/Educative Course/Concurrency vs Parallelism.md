These notes are mainly based on [this](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/qAVrEQEw6rR) Educative course page.

## Concurrency
* Is when $\geq 2$ tasks[^fn1] run in overlapping time periods.
* They're not literally running at the same instance.
* It's about *dealing with* multiple things at once.
* An example is a multitasking OS. It quickly switches between tasks, in a way that makes it seem like they're executing simultaneously.

## Parallelism
* Is when $\geq 2$ tasks literally run at the same time.
* It's about *doing* multiple things at once.
* This is usually aided by hardware.
* An example is an OS running on a multicore processor.

A useful point to note is that:

* Concurrency $\centernot\implies$ parallelism.
* Parallelism $\implies$ concurrency. Specifically, parallelism is a specific case of concurrency where task execution actually overlaps.
* In other words, parallel programs are a **subset** of concurrent programs.

[^fn1]: In this context, a task is just any unit of work that a computer program needs to accomplish. For example, it could refer to a thread, or an entire process.