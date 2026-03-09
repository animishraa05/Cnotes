
# 09. The Transport Layer

## Introduction to the Transport Layer

The transport layer is the fourth layer of the OSI model. It is responsible for providing reliable, end-to-end communication between applications. The transport layer takes the data from the application layer and segments it into smaller units called segments. It then sends these segments to the network layer for transmission. The transport layer is also responsible for reassembling the segments at the destination and ensuring that all the data is received correctly and in order.

## Functions of the Transport Layer

- **Service-Point Addressing:** The transport layer is responsible for delivering the data to the correct process on the destination computer. This is done by using a port number. A port number is a unique address that is assigned to each process.
- **Segmentation and Reassembly:** The transport layer is responsible for breaking up the data from the application layer into smaller segments and for reassembling the segments at the destination.
- **Connection Control:** The transport layer can provide either a connection-oriented or a connectionless service. A connection-oriented service establishes a connection before sending the data, while a connectionless service does not.
- **Flow Control:** The transport layer is responsible for regulating the flow of data so that a fast sender does not overwhelm a slow receiver.
- **Error Control:** The transport layer is responsible for ensuring that all the data is received correctly and in order. This is done by using error detection and correction mechanisms.

## Key Protocols of the Transport Layer

- **Transmission Control Protocol (TCP):** TCP is a connection-oriented protocol that provides reliable, ordered, and error-checked delivery of a stream of octets between applications. TCP is the most common transport layer protocol.
- **User Datagram Protocol (UDP):** UDP is a connectionless protocol that provides a simple, but unreliable, datagram service. UDP is used for applications that do not require reliable delivery, such as video streaming and online gaming.

## TCP vs. UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable | Unreliable |
| Ordering | Ordered | Unordered |
| Error Checking | Yes | No |
| Speed | Slower | Faster |
| Overhead | Higher | Lower |
| Use Cases | Web browsing, email, file transfer | Video streaming, online gaming, DNS |

## Ports

A port is a 16-bit number that is used to identify a specific process on a computer. A port number is used in conjunction with an IP address to form a socket. A socket is a unique identifier for a process on a network.

There are two types of ports:

- **Well-Known Ports:** These are ports that are assigned to specific services. For example, port 80 is used for HTTP, and port 25 is used for SMTP.
- **Ephemeral Ports:** These are ports that are assigned dynamically to client applications. Ephemeral ports are in the range of 1024 to 65535.
