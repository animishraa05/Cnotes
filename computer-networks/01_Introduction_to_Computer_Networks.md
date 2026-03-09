---
id: 01_Introduction_to_Computer_Networks
aliases: []
tags: []
---


# 01. Introduction to Computer Networks

## What is a Computer Network?

A computer network is a collection of interconnected autonomous computers. Two computers are said to be interconnected if they are able to exchange information. The connection need not be via a copper wire; fiber optics, microwaves, infrared, and communication satellites can also be used. Networks come in many sizes, shapes and forms. They are usually connected together to make larger networks, with the Internet being the most well-known example of a network of networks.

## Uses of Computer Networks

### Business Applications

Many companies have a substantial number of computers. For example, a company might have a computer for each employee and use them to design products, produce documents, and keep track of inventory and accounts. In the past, these computers were largely isolated from one another, but now they are connected into a network.

- **Resource Sharing:** The goal is to make all programs, equipment, and especially data available to anyone on the network without regard to the physical location of the resource and the user.
- **High Reliability:** A file can be replicated on two or three machines, so if one of them is unavailable (e.g., due to a hardware crash), the other copies could be used.
- **Saving Money:** Small computers have a much better price/performance ratio than large ones. Mainframes are roughly a factor of 10 faster than personal computers, but they cost a thousand times more. This imbalance has led many companies to a model in which personal computers are used for interactive work and a central machine, the server, is used for storing shared data. This arrangement is called the client-server model.
- **Scalability:** The ability to increase system performance gradually as the workload grows by just adding more processors.
- **Communication Medium:** A computer network can provide a powerful communication medium among employees. Email, video conferencing, and other forms of communication are widely used.

### Home Applications

- **Access to Remote Information:** This includes surfing the World Wide Web for information or entertainment, and accessing social media.
- **Person-to-Person Communication:** This includes email, instant messaging, and video calls.
- **Interactive Entertainment:** This includes online gaming, video on demand, and interactive television.
- **Electronic Commerce:** This includes online shopping and banking.

## Network Hardware

There are two types of transmission technology that are in widespread use:

- **Broadcast Links:** Broadcast networks have a single communication channel that is shared by all the machines on the network. Packets (short messages) sent by any machine are received by all the others. An address field within the packet specifies the intended recipient. Upon receiving a packet, a machine checks the address field. If the packet is intended for it, it processes the packet; if it is intended for some other machine, it is just ignored.
- **Point-to-Point Links:** Point-to-point networks consist of many connections between individual pairs of machines. To go from the source to the destination, a packet on this type of network may have to first visit one or more intermediate machines.

### LAN, MAN, WAN

- **Local Area Networks (LANs):** These are privately-owned networks within a single building or campus of up to a few kilometers in size. They are widely used to connect personal computers and workstations in company offices and factories to share resources (e.g., printers) and exchange information.
- **Metropolitan Area Networks (MANs):** A MAN is basically a bigger version of a LAN and normally uses similar technology. It might cover a group of nearby corporate offices or a city and might be either private or public.
- **Wide Area Networks (WANs):** A WAN spans a large geographical area, often a country or continent. It contains a collection of machines (hosts) intended for running user (i.e., application) programs.

## Network Software

### Protocol Hierarchies

To reduce their design complexity, most networks are organized as a stack of layers or levels, each one built upon the one below it. The number of layers, the name of each layer, the contents of each layer, and the function of each layer differ from network to network. The purpose of each layer is to offer certain services to the higher layers, shielding those layers from the details of how the offered services are actually implemented.

### Design Issues for the Layers

- **Addressing:** Every layer needs a mechanism for identifying senders and receivers.
- **Error Control:** Error detection and error correction are necessary to ensure reliable communication.
- **Flow Control:** A fast sender can drown a slow receiver with data. Some mechanism must be employed to let the sender know how much data the receiver can handle.
- **Multiplexing/Demultiplexing:** A layer may need to combine several unrelated protocols into a common one for transmission, and then separate them at the other end.
- **Routing:** When there are multiple paths between source and destination, a route must be chosen.

### Connection-Oriented vs. Connectionless Service

- **Connection-Oriented Service:** This service is modeled after the telephone system. To talk to someone, you pick up the phone, dial the number, talk, and then hang up. Similarly, in a network, the service user first establishes a connection, uses the connection, and then releases the connection.
- **Connectionless Service:** This service is modeled after the postal system. Each message (letter) carries the full destination address, and each one is routed through the system independent of all the others.

## Reference Models

### The OSI Reference Model

The Open Systems Interconnection (OSI) model is a conceptual framework that standardizes the functions of a telecommunication or computing system in terms of seven abstraction layers.

1.  **Physical Layer:** The physical layer is concerned with transmitting raw bits over a communication channel.
2.  **Data Link Layer:** The main task of the data link layer is to transform a raw transmission facility into a line that appears free of undetected transmission errors to the network layer.
3.  **Network Layer:** The network layer controls the operation of the subnet. A key design issue is determining how packets are routed from source to destination.
4.  **Transport Layer:** The basic function of the transport layer is to accept data from above, split it up into smaller units if need be, pass these to the network layer, and ensure that the pieces all arrive correctly at the other end.
5.  **Session Layer:** The session layer allows users on different machines to establish sessions between them.
6.  **Presentation Layer:** The presentation layer is concerned with the syntax and semantics of the information transmitted.
7.  **Application Layer:** The application layer contains a variety of protocols that are commonly needed by users.

### The TCP/IP Reference Model

The TCP/IP model is a more practical model that is used in the real world. It consists of four layers:

1.  **Link Layer:** The link layer (or network interface layer) is the lowest layer in the TCP/IP model. It describes what links must do to meet the needs of this connectionless internet layer.
2.  **Internet Layer:** The internet layer's job is to permit hosts to inject packets into any network and have them travel independently to the destination.
3.  **Transport Layer:** The transport layer is designed to allow peer entities on the source and destination hosts to carry on a conversation, just as in the OSI transport layer. Two end-to-end transport protocols have been defined here: TCP and UDP.
4.  **Application Layer:** The application layer contains all the higher-level protocols. Some of them are the virtual terminal (TELNET), file transfer (FTP), and electronic mail s(sSMTP).
