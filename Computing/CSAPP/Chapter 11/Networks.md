Clients and servers often run on separate hosts (i.e. machines) and communicate using the hardware and software of a computer **network**. Networks are really complicated, and we only scratch the surface in this chapter.

To a host, a network is just another I/O device that serves as a source/sink of data:

![](_attachments/Screenshot%202023-09-05%20at%2020.08.20.png)

This subsection is quite wordy, but worth understanding. I've split the key content into small paragraphs below.

> An **adapter** provides the physical interface to the network. Data received from the network are copied into main memory via [DMA](1%20-%20Hardware%20Basics#^7bdae2). 

> Physically, a network is a hierarchical system organised by geographical proximity. At the lowest level is a **Local Area Network (LAN)**. A LAN spans a building or a campus. The most popular LAN technology is **Ethernet**. An **Ethernet segment** is shown below:

![](_attachments/Screenshot%202023-09-05%20at%2020.12.31.png)

> An Ethernet segment consists of some wires and a small box, called a hub. Ethernet segments typically span small areas, such as a room or a floor in a building. Each wire has the same bandwidth. One end is attached to an adapter on a host; the other is attached to a **port** on the hub. A hub copies every bit that it receives on each port to every other port. Thus, every host sees every bit.

> [!INFO]
> In this context, a port refers to a specific point on a hub/bridge where a wire is attached. In more general networking terminology, a port can also refer to a virtual endpoint for network communications in software.

> Each ethernet adapter has a globally unique 48-bit address, stored in nonvolatile memory on the adapter. A host can send a chunk of bits called a **frame** to any other host on the segment. Each frame includes some fixed number of **header** bits that identify the source and destination bits of the frame, and the frame length. It then contains a [payload](Dynamic%20Memory%20Allocation#^099f6d) of data bits. Every host adapter sees the frame, but only the destination host reads it.

^051a6e

> Multiple ethernet segments can be connected into larger LANs, called **bridged ethernets**:

![](_attachments/Screenshot%202023-09-05%20at%2020.22.48.png)

> Bridged ethernets can span entire buildings or campuses. They use a set of wires, hubs, and small boxes called **bridges**. Note that the bandwidths of the wires can be different.

> Bridges make better use of the available wires than hubs. Using a clever distributed algorithm, they learn which hosts are reachable from which ports and then selectively copy frames from one port to another only when it is necessary. For instance, if host `A` sends a frame to host `B`, which is on the same segment, then bridge `X` will throw away the frame when it arrives at its port. However, if `A` sends a frame to `C`, then bridge `X` will copy the frame only to the port connected to bridge `Y`, which will then copy the frame only to the port connected to `C`'s segment.

> Conceptually, we can draw a LAN in a way that abstracts away the hubs & bridges:

![](_attachments/Screenshot%202023-09-05%20at%2020.30.59.png)

> At a higher level in the hierarchy, multiple incompatible LANs can be connected by specialised **routers** to form an **internet** (interconnected network):

![](_attachments/Screenshot%202023-09-05%20at%2020.32.42.png)

> Each router has an adapter (port) for each network that it is connected to. Routers can also connect high-speed phone connects, which are examples of networks called **WANs (Wide Area Networks)**. In general, router can be used to build internets from arbitrary collections of LANs and WANs.

> The crucial property of an internet is that it can consist of different LANs and WANs with radically different and incompatible technologies. 

> This is done using a layer of **protocol software** running on each host and router that smooths out the differences between networks. This software implements a **protocol** that governs how hosts and router cooperate. The protocol must provide two basic capabilites:

1. **Naming Scheme**: Different LAN technologies have different ways of assigning addresses to hosts. The internet protocol sorts this by defining a uniform format for host addresses. Each host is then assigned at least one of these **internet addresses**.
2. **Delivery Mechanism**: Different networking technologies have different ways of encoding bits, and packaging them into frames. The internet protocol defines a uniform way to bundle data into discrete chunks called **packets**. A packet consists of a **header**, and **payload** (as [above](#^051a6e)).

> Figure 11.7 shows an example of how hosts and routers use the internet protocol to transfer data between LANs. The example internet consists of two LANs connected by a router. A client running on host `A` sends a sequence of bytes to a server running on host `B`:

![](_attachments/Screenshot%202023-09-05%20at%2020.40.38.png)

> There are 8 basic steps:

1. The client invokes a syscall that copies data from the client's virtual address space into a kernel buffer.
2. The protocol software on host `A` creates a LAN1 frame by appending a LAN1 frame header, and internet header, to the data. The LAN1 frame header is addressed to the router. The internet header is addressed to host `B`. It then passes the frame to the adapter. Notice that the *payload* of the LAN1 frame is an internet packet, whose *payload* is the actual user data. This **encapsulation** is one of the fundamental ideas in networking.
3. The LAN1 adapter copies the frame to the network.
4. When the frame reaches the router, the router's LAN1 adapter reads it from the wire and passes it to the protocol software.
5. The router fetches the destination internet address from the internet packet header. It uses this as an index into a routing table to determine where to forward the packet, which in this case is LAN2. The router strips off the old LAN1 frame header, and prepends a new LAN2 frame header addressed to host `B`. It passes the resulting frame to the adapter.
6. The router's LAN2 adapter copies the frame to the network.
7. Host `B`'s adapter reads the frame and passes it to the protocol software.
8. The protocol software on host `B` strips off the packet header and frame header. It then copies the resulting data into the server's virtual address space when the server invokes a syscall that reads that data.

> We are glossing over many difficult issues here. Whwat if different networks have different maximum frame sizes? What if a packet gets lost? How are routers informed when the network topology changes?
> Nonetheless, our example captures the essence of the internet idea, and encapsulation is key.
