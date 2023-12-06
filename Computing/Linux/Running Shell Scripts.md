Suppose we run a shell script in Linux, i.e. we do something like `./script.sh`. What happens next?

According to [ChatGPT](https://chat.openai.com/c/6b368b77-b84b-45f7-b72c-2461c858ef7a), the system follows these general steps:

1. Shell recognises the request to execute file at the given path.
2. Shell launches a child process. This is typically another instance of the same shell type, but could be different if the script's [shebang](Shebangs.md) dictates this.
3. Child shell process then interprets the script line by line. For each command it encounters, it may launch additional child processes.

### Sourcing a Shell Script
If we choose to [source](https://tldp.org/HOWTO/Bash-Prompt-HOWTO/x237.html) the script, for instance doing `source script.sh` or `. script.sh`, then we get different behaviour. The key point is that:

> Instead of spawning a child process, the commands in the script are executed by the current shell process.

This means any variables or functions that are defined in the script will remain in your current shell environment, even after the script has finished running. To emphasise: no new processes are created to run the script, although commands within the script may still spawn their own child processes.

### Case Study: Reparenting
Suppose we have the following shell script:

```bash
#! /bin/bash
sleep 1000
```

and we run it as follows:

```bash
script.sh
```

What is the PPID of the `sleep` process? Clearly, it is the shell process that spawned it. This looks something like:

![](_attachments/Screenshot%202023-05-22%20at%2017.02.08.png)

where `./aladdin/scrapers/youtube/channel_downloader/test.sh` is the name of our script. If we source the script instead:

```bash
. script.sh
```

then we get:

![](_attachments/Screenshot%202023-05-22%20at%2017.04.02.png)

As expected, the parent of the `sleep` process is the shell.
But what happens if we alter the shell script to `sleep` in the background?

```bash
#! /bin/bash
sleep 1000 &
```

If we source the script, then nothing changes:

![](_attachments/Screenshot%202023-05-22%20at%2017.05.32.png)

But if we run the script in the ordinary fashion, then we see something strange:

![](_attachments/Screenshot%202023-05-22%20at%2017.06.45.png)

The `sleep` process appears to have the init process as its parent. This is confirmed when we run `ps -f`:

![](_attachments/Screenshot%202023-05-22%20at%2017.07.30.png)

What's going on here? 
In this case, the `sleep` process is ongoing but its parent process (the one that runs the shell script) finishes. The `sleep` process loses its parent. Such processes are called [orphan processes](https://en.wikipedia.org/wiki/Orphan_process). 
Linux collects orphan processes, and sets their parent process to the init process. This process is called [reparenting](https://stackoverflow.com/questions/6476452/process-re-parenting-controlling-who-is-the-new-parent).