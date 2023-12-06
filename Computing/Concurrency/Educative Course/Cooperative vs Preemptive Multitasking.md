These notes are mainly based on [this](https://www.educative.io/courses/python-concurrency-for-senior-engineering-interviews/Y5G6Q45Ny4n) Educative page.

A system can achieve concurrency by employing one of the following multitasking models:

* [Preemptive](4%20-%20Processes#^4966f8) multitasking
* Cooperative multitasking

## Preemptive Multitasking
Aka time-sharing.

The OS preempts a program to allow another waiting task to run on CPU. The OS scheduler decides which thread/program gets to use the CPU next.

## Cooperative Multitasking
Aka non-preemptive multitasking.

Programs voluntarily give up control to the scheduler so that another program can run. The OS scheduler has no say in how long a process/thread runs for.

## Nowadays
Whilst Windows and macOS used to implement cooperative multiasking, modern OSs (including Windows, macOS, and Linux) all use preemptive multitasking.


