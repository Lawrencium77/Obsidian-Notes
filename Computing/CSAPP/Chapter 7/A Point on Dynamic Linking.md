These notes are relevant to [Chapter 7 - Linking](Chapter%207%20-%20Linking.md) but based on [this](https://stackoverflow.com/questions/51209319/why-does-g-detect-undefined-reference-when-dynamically-linking?noredirect=1&lq=1) Stack Overflow page.

## Question
When doing [load-time dynamic linking](Dynamic%20Linking%20with%20Shared%20Libraries.md), the symbols from the shared object are resolved at run time. However, `g++` can throw an "undefined reference" error at compile time if a symbol doesn't have a definition. So:

1. Why does `g++` require the shared object when dynamically linking? 
2. Why does `g++` attempt to resolve symbols when dynamically linking? It's possible to use a different `.so` file at link time and run time, which could cause a crash.

## Answer
In dynamic linking, the linker checks for the existence of symbols at link time to ensure that all necessary symbols can be located. This is a convention, and is for your convenience - it means that we get immediate feedback when there are undefined symbols.
But the actual symbol resolution still happens at runtime.

