This section looks at how data and control interact.

```toc
```

## Understanding Pointers
Here, the authors highlight some key principles of pointers. I just picked out the parts that weren't immediately familiar.

* The special pointer type, `void *`, represents a generic pointer. 
* **Pointers can also point to functions**. This provides capability for storing and passing references to code. For instance, if we have a function defined as

```C
int fun(int x, int* p);
```

then we declare and assign a pointer `fp` to this function as:

```C
int (*fp)(int, int *);
fp = fun;
```

We can then invoke the function using this pointer:

```C
int y = 1;
int result = fp(3, &y)
```

The value of a function pointer is the address of the first instruction in the machine-code representation of that function.

> [!INFO]
> #### More on Function Pointers
> The syntax for declaring function pointers is a bit weird. For a declaration like
> 
> ```C
> int (*fp)(int*);
> ```
> it helps to read starting from the inside (the `fp`) and working outward. We see that `fp` is a pointer, to a function that takes a single `int*` as argument. Finally, we see that it has return type `int`.
> The parentheses around `*fp` are required because otherwise, the declaration
> 
> ```C
> int *f(int*)
> ```
> 
> would be read as
> 
> ```C
> (int *) f(int *)
> ```
> i.e. a declaration of a function `f` that has an `int *` argument.
> 

## Out-of-Bounds Memory References and Buffer Overflow
C does not perform any bounds checking for array references. In addition, local variables are stored on the stack, along with information such as saved register values and return addresses. This combination can lead to nasty bugs.

A common source of memory corruption is **buffer overflow**. This happens when a program writes more data to a block of memory than it has allocated. The behaviour that this causes can be odd; for instance, an overflow might end up writing to the value of the return pointer, meaning the `ret` instruction will jump to a totally unexpected location.

A more pernicious use of buffer overflow is to get a program to perform a function that it would otherwise be unwilling to do. This is a common method in attacking the security of a system over a computer network. Typically, a program is fed with a string that contains the byte encoding of some executable code, called the **exploit code**, plus some extra bytes that overwrite the return address with a pointer to the exploit code. The effect of executing the `ret` instruction is to jump to the exploit code.

In one form of attack, the exploit code then uses a syscall to start up a shell program, providing the attacker with a range of OS functions. In another for, the exploit code performs some otherwise unauthorised task, repairs the damage to the stack, and executes `ret` a second time, causing an (apparently) normal return to the caller.

## Thwarting Buffer Overflow Attacks
Modern compilers and OSs implement mechanisms to make it more difficult to mount buffer overflow attacks. This section presents mechanisms that are provided by recent versions of `gcc` for Linux.

### Stack Randomisation
In order to insert exploit code into a system, the attacker needs to inject both the code, and a pointer to this code as part of the attack string. Generating this pointer requires knowing the stack address where the string will be located. Historically, the stack addresses for a program were highly predictable. For all systems running the same combination of programs with the same OS, the stack locations were stable across machines. For example, if an attacker could determine the stack addresses used by a common Web server, it could devise an attack that would work on many machines.

The idea of **stack randomisation** is to make the position of the stack vary between program runs. Thus, even if many machines are running identical code, they would all be using different stack addresses. This is implemented by allocating a random amount of space between `0` and `n` bytes on the stack at the start of a program. The allocation range `n` needs to be large enough to get sufficient variation in stack addresses, yet small enough that it doesn't waste too much memory.

Stack randomisation has become standard practice in Linux systems. It is one of a large classes of techniques called **Address-Space Layout Randomisation (ASLR)**. With ASLR, different parts of the program (program code, library code, global variables, and heap data) are loaded into different regions of memory each time a program is run.

However, a persistent attacker can overcome randomisation by brute force, repeatedly attempting attacks with different addresses. A common trick is to include a long sequence of `nop` (no-op) instructions before the actual exploit code. As long as the attack can guess an address somewhere within this sequence, the program will run through the sequence and hit the exploit code. The common term for this sequence is a "nop sled".

### Stack Corruption Detection
A second line of defence is to be able to detect when a stack has been corrupted. In C, there is no reliable way to prevent writing beyond the bounds of an array. Instead, the program can attempt to detect when such a write has occurred before it can have any harmful effects.

Recent versions of `gcc` incorporate a mechanism called a **stack protector** into the generate code, to detect buffer overruns. The idea is to store a special ***canary*** value in the stack frame between any local buffer and the rest of the stack state, as illustrated in Figure 3.42:

![](_attachments/Screenshot%202023-10-12%20at%2020.27.17.png)

This canary value (aka a ***guard value***) is generated randomly each time the program runs, meaning there is no easy way for an attacker to determine what it is. Before restoring the register state and returning from the function, the program checks if the canary value has been altered by an operation of this function (`echo`), or one it has called. If so, the program aborts with an error.

Recent versions of `gcc` try to determine whether a function is vulnerable to a stack overflow and insert this type of overflow detection automatically. 

> [!TODO]
> Ankied up to here.

### Limiting Executable Code Regions
In typical programs, only the portion of memory holding code generated by the compiler needs to be executable. The rest can just allow reading/writing. Therefore, the stack can be marked as readable and writable, but **not executable**, meaning an attacker couldn't execute any code they placed on the stack.

## Supporting Variable-Size Stack Frames
So far, we have examined procedures for which the compiler can determine in advance the amount of space that must be allocated for their stack frames. Some functions, however, require a variable amount of local storage. For instance, they can declare a local array of variable size.

The code below gives an example of a function containing a variable-size array. It declares a local array `p` of `n` pointers, where `n` is given by the first argument. This requires `8 * n` bytes on the stack. The point is that `n` can vary between function calls. The compiler therefore cannot determine how much space it must allocate for the function's stack frame. In addition, the program generates a reference to the address of the local variable `i`, and this variable must also be stored on the stack.

```C
long vframe(long n, long idx, long *q)  {
    long i;

    long *p[n];
    p[0] = &i;
    for (i = 1; i < n; i++)
        p[i] = q;
    
    return *p[idx];
}
```

To manage a variable-size stack frame, x86-64 code uses register `%rbp` to serve as a **frame pointer**. When using a frame pointer, the stack frame is organised as shown in Figure 3.44:

![](_attachments/Screenshot%202023-10-13%20at%2022.19.40.png)

The key idea is that `%rbp` marks the beginning of the current procedure's stack frame. This allows for a consistent way to access function arguments and local variables, regardless of how the stack grows/shrinks during the function's execution. Local variables are referenced as offsets relative to this value. Clearly, we must save the previous version of `%rbp` on the stack, since it is a callee-saved register. 

Figure 3.43(b) shows portions of code that `gcc` generates for function `vframe`:

![](_attachments/Screenshot%202023-10-13%20at%2022.22.55.png)

The code starts by pushing the current value of `%rbp` onto the stack and setting `%rbp` to point to this stack position. Next, it allocates 16 bytes on the stack, the first 8 of which are used to store local variable `i`, and the second 8 of which are unused. Then it allocates space for array `p`. By the time it reaches line 11, it has allocated at least `8 * n` bytes on the stack.

At the end of this function, the frame pointer is restored to its previous value using the `leave` instruction. It sets the stack pointer to the position of the saved value of `%rbp`, and then this value is popped from the stack into `%rbp`. This has the effect of deallocating the entire stack frame.





