Linux shells provide *I/O redirection* operators that allow users to associate standard input and output with disk files. For instance:

```bash
ls > foo.txt
```

So how does this work?

On a high level, the `>` and `<` operators copy the [descriptor table](Sharing%20Files#^c6d1a0) entry for the target file (`foo.txt`) to that of stdin/stdout, thus overwriting the previous contents of `fd[0]` or `fd[1]`.

As a diagram example, suppose we have:

![](_attachments/Screenshot%202023-06-04%20at%2013.37.59.png)

After redirecting stdout, we then have:

![](_attachments/Screenshot%202023-06-04%20at%2013.49.40.png)



