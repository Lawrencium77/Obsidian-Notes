Every network application is based on the [client-server model](https://en.wikipedia.org/wiki/Client%E2%80%93server_model). With this model, an application consists of a **server process** and one or more **client processes**. A server manages some **resource**, and it provides a **service** for its clients by manipulating that resource. For example, a Web server manages a set of disk files that it retrieves and reads on behalf of clients. 

The fundamental operation in the client-server model is the **transaction**:

![](_attachments/Screenshot%202023-06-29%20at%2017.01.24.png)

As shown in Figure 11.1, it consists of four steps:

1. When a client needs a service, it initiates a transaction by sending a **request** to the server. For instance, when a Web browser needs a file, it sends a request to a Web server.
2. The server receives the request, interprets it, and manipulates its resources as appropriate. For example, when a Web server receives a request from a browser, it reads a disk file.
3. The server sends a **response** to the client and then waits for the next request. For example, the Web server sends the file back to a client.
4. The client receives and processes the response. For instance, after a Web browser receives a page from the server, it displays it on the screen.

> [!INFO]
> It's important to appreciate that clients and servers are *processes* and not machines, or *hosts* as they're often called in this context. A single host can run multiple clients and servers concurrently; and a client-server transaction can happen on the same or different hosts. 

