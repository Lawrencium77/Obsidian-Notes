Up to this point in our tour of systems, we have treated a system as an isolated collection of hardware and software. In practice, modern systems are often linked to other systems by networks. 

From the point of a view of an individual system, the network is seen as just another I/O device, as shown in Figure 1.14. When the system copies a sequence of bytes from main memory to the network adapter, the data flow across the network to another machine. Similarly, the system can read data sent from other machines and copy these data to its main memory.

![](_attachments/Screenshot%202022-05-18%20at%2015.48.34.png)

Returning to our  `hello`  example, we could use the [*telnet*](https://en.wikipedia.org/wiki/Telnet)application to run  `hello`  on a remote machine. Suppose we use a telnet **client** running on our local machine to connect to a telnet **server** on a remote machine. After we log in to the remote machine and run a shell, the remote shell is waiting to receive an input command. From this point, running  `hello`  remotely involves the five basic steps shown in Figure 1.15.

![](_attachments/Screenshot%202022-05-18%20at%2015.51.59.png)

After we type  `hello`  to the telnet client and hit the `enter` key, the client sends the string to the telnet server. After the telnet server receives the string from the network, it passes it along to the remote shell program. Next, the remote shell runs the  `hello`  program and passes the output back to the telnet server. Finally, the telnet server forwards the output string across the network to the telnet client, which prints the output string on our local terminal.

This type of exchange between clients and servers is typical of all network applications. It's studied in Chapter 11.
