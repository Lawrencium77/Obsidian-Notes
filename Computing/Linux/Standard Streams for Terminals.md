Ed had a couple of fun tricks in Linux. One of them is that we can write to all terminals attached to a device by sshing onto that machine, and then using the `wall` command.

In learning about this, there were another couple of fun bits.

First, the directory `/proc/$$/fd` contains all the file descriptors for some [process](The%20Linux%20Kernel/4%20-%20Processes.md) (whose PID is represented by `$$).

Second, the standard streams for each [shell](4%20-%20Processes#^e21269) process seem to point to a file in `/dev/pts`. Seeing what's in this directory:

```bash
❯ pwd
/dev/pts
❯ ls
0  11  13  4  5  ptmx
```

To be clear: the stdin/out/err for each shell process are all the same file. And its contained in this directory.

If you do something like:

```bash
echo "hello" > /dev/pts/2
```

then this message will appear in the terminal corresponding to `/dev/pts/2`.

> [!QUESTION]
> There is a lot more detail regarding the `/dev/pts` folder. Most of which I didn't understand. It'd be nice to come back round to it at some point.




