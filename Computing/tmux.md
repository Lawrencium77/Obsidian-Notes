I was thinking a little bit about how `tmux` works. According to the Linux [man page](https://man7.org/linux/man-pages/man1/tmux.1.html#:~:text=tmux%20is%20a%20terminal%20multiplexer,and%20displays%20it%20on%20screen.):

> [!QUOTE]
> In `tmux`, a session is displayed on screen by a client and all sessions are managed by a single server.  The server and each client are separate processes which communicate through a socket in `/tmp`.

So we've another example of the [The Client-Server Programming Model](CSAPP/Chapter%2011/The%20Client-Server%20Programming%20Model.md).
Can we see this in action? Answer: yes!

Let's begin by just running the `tmux` command. After doing this, we spin up a `tmux` server and are presented with an empty window:

![](_attachments/Screenshot%202023-06-30%20at%2022.18.31.png)

What processes are running at this point? Answer: one for the server, and one for the client:

![](_attachments/Screenshot%202023-06-30%20at%2022.21.05.png)
![](_attachments/Screenshot%202023-06-30%20at%2022.20.44.png)

When we open more tmux windows, we see that the server spawns more child processes. But there is still only one server (and one client):

![](_attachments/Screenshot%202023-06-30%20at%2022.22.29.png)

This is even true when we run `tmux` for a second time, without killing the first window:

![](_attachments/Screenshot%202023-06-30%20at%2022.23.18.png)

It is possible to create multiple client processes. This is done by attaching to `tmux` for a second time from the same machine, e.g:

![](_attachments/Screenshot%202023-06-30%20at%2022.27.30.png)

(I'm not totally sure why the second client process here is `tmux` and not `tmux attach`).

> [!SUMMARY]
   A nice summary of this is given by ChatGPT:   
> 
> When you start `tmux`, it creates a single server process. This server process manages all of the `tmux` sessions, windows, and panes that you create during your usage of `tmux`. The server is essential as it maintains the state of all `tmux` sessions. It allows sessions to persist in the background even when no clients are connected to them.
> 
> A client is any terminal instance that attaches to a `tmux` session. Multiple clients can attach to the same session. To create multiple clients, we start `tmux` sessions from multiple terminals.











