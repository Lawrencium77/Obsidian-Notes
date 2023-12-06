> [!INFO]
> #### Internet versus internet
> CSAPP uses lowercase *internet* to denote the general concept, and uppercase *Internet* to denote a specific implementation - namely, the global IP Internet.

The **Global IP Internet** is the most famous implementation of an internet. Figure 11.8 shows the basic hardware and software organisation of an Internet client-server application:

![](_attachments/Screenshot%202023-09-05%20at%2020.55.36.png)

Each Internet host runs software that implements the **TCP/IP** protocol, which is supported by almost every modern computer system. Internet clients and servers communicate using a mix of [Sockets Interface](The%20Sockets%20Interface.md) functions and Unix I/O functions. 

TCP/IP is actually a family of protocols. For example, **IP** provides the basic naming scheme and a delivery mechanism to send packets between hosts. The IP mechanism is unreliable in that it makes no effort to recover lost packets. **UDP** (Unreliable Datagram Protocol) extends IP slightly, so that packets can be transferred between processes rather than hosts. **TCP** is a complex protocol that builds on IP to provide reliable bidirectional **connections** between processes. 

To simplify our discussion, we'll treat TCP/IP as a single monolithic protocol. We won't discuss its inner workings. We won't discuss UDP.

From a programmer's perspective, we can think of the Internet as a worldwide collection of hosts with the following properties:

> 1. The set of hosts is mapped to a set of 32-bit **IP addresses**.
> 2. The set of IP addresses is mapped to a set of identifiers called **Internet domain names**.
> 3. A process on one host can communicate with a process on any other host over a **connection**.

The following sections discuss these ideas in more detail.

### IP Addresses
An IP address is an unsigned 32-bit integer. Because internet hosts can have different byte orderings, TCP/IP defines a uniform **network byte order** (big-endian) for any integer data item that is carried across the network in a packet header (such as an IP address). 

IP addresses are typically presented to humans in a form called **dotted-decimal notation**. Each byte is represented by its decimal value and separated from the other bytes by a full stop. For instance:

```
128.2.192.242
```

is the dotted-decimal representation of the address `0x8002c2f2`. On Linux systems, the `hostname -i` command shows the dotted decimal address of your own host:

```
linux> hostname -i
128.2.210.175
```

### Internet Domain Names
> [!INFO]
> Look at [this](https://wizardzines.com/comics/dns-hierarchy/) cartoon describing the DNS hierarchy for a fun take.

Internet clients and servers use IP addresses when they communicate. But large integers are difficult for people to remember, so the Internet also defines a separate set of more human-friendly **domain names**, as well as a mechanism that maps the set of domain names to the set of IP addresses.

A domain name is a sequence of words separated by full stops:

```
whaleshark.ics.cs.cmu.edu
```

The set of domain names forms a hierarchy, and each domain name encodes its position in the hierarchy. An example is the easiest way to understand this. Consider Figure 11.10:

![](_attachments/Screenshot%202023-09-05%20at%2021.16.49.png)

The hierarchy is represented as a tree. The nodes represent domain names that are formed by the path back to the root. Subtrees are referred to a **sub-domains**. The first level in the hierarchy is the unnamed root node. The next level is a collected of **first-level domain names** that are defined by a nonprofit org called **ICANN (Internet Corporation for Assigned Names and Numbers)**. Common first-level domain names include `com`, `gov`, and `org`.

At the next level are **second-level** domain names. These are assigned on a first-come first-serve basis by various authorised agents of ICANN. Once an organisation has received a second-level domain name, it's free to create any other new domain name within its subdomain.

The Internet defines a mapping between the set of domain names and the set of IP addresses. Until 1998, this mapping was maintained manually in a single text file called `HOSTS.TXT`. Since then, the mapping has been maintained in a distributed worldwide database called **DNS (Domain Name System)**. Conceptually, you can think of it as a large table where one column lists domain names and the corresponding column lists IP addresses. Each entry in the table is called a **host entry**.

We can explore some of the properties of the DNS mappings with Linux's `nslookup` program, which displays the IP addresses associated a domain name.

Each Internet host has the locally defined name `localhost`, which always maps to the **loopback address** `127.0.0.1`:

```
linux> nslookup localhost
Address: 127.0.0.1
```

> [!INFO]
> Note that we've reformatted the output of `nslookup` for readability.

The `localhost` name provides a portable way to reference clients and servers that are running on the same machine. We can use `hostname` to determine the real domain name of our local host:

```
linux> hostname
gpu010.grid.speechmatics.io
```

In the simplest case, there is a one-to-one mapping between a domain name and an IP address:

```
linux> nsloopkup gpu010.grid.speechmatics.io
Address: 172.21.1.20
```

However, in some cases, multiple domain names are mapped to the same IP address:

```
linux> nslookup cs.mit.edu
Address: 18.25.0.23

linux> nslookup eecs.mit.edu
Address: 18.25.0.23
```

And in the most general case, multiple domain names are mapped to the same set of **multiple** IP addresses. We also notice that some valid domain names are not mapped to any IP address:

```
linux> nslookup edu
*** Can't find edu: No answer
```

### Internet Connections
Internet clients and servers communicate by sending and receiving packets via **connections**. A connection connects a **pair of processes**.  They are **full duplex**, meaning that data can flow in both directions. And it is **reliable** - barring some catastrophic failure, the packets sent by the source process are eventually received by the destination process in the same order they were sent.

A **socket** is an end point of a connection. Each socket has a corresponding **socket address**, which consists of an Internet address and a 16-bit integer **port**[^fn1]. This address is denoted with the notation `address:port`.

> The key idea here is that while an IP address identifies a machine, a socket identifies a specific process on that machine. When a packet arrives at a machine, the OS checks the port number and forwards it to the correct application listening on that port. 
> This ability to direct data to multiple applications is called using different ports on the same IP address is called **port multiplexing**. 

The port in the client's socket address is assigned automatically by the kernel when the client makes a connection request, and is called an **ephemeral (temporary) port**. 
In contrast, the port in the server's socket address is typically a **well-known port** that's permanently associated with some service. For instance, Web servers typically use port `80`. Associated with each service with a well-known port is a corresponding **well-known service name**. For instance, the well-known name for the Web service is `http`. The mapping between well-known names and well-known services is kept in `/etc/services`.

A connection is uniquely identified by the **socket address** of its two end points. The pair of socket addresses is called a **socket pair** and is denoted by:

```
(clientaddr:clientport, serveraddr:serverport)
```

To be clear: a socket is a combination of an IP address and port number. Figure 11.11 shows a connection between a Web client and a Web server:

![](_attachments/Screenshot%202023-09-20%20at%2018.42.04.png)

[^fn1]: These software ports have no relation to the hardware ports in network switches and routers.