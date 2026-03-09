
# 05. The TCP/IP Model

## Introduction to the TCP/IP Model

The TCP/IP model is a concise and practical model for computer networking. It is the model that is used in the real world, and it is the model that the Internet is based on. The TCP/IP model is also known as the Internet Protocol Suite. It was developed by the United States Department of Defense Advanced Research Projects Agency (DARPA) in the 1970s.

The TCP/IP model is a four-layer model, in contrast to the seven-layer OSI model. The four layers of the TCP/IP model are the Link Layer, the Internet Layer, the Transport Layer, and the Application Layer.

## The Four Layers of the TCP/IP Model

### 1. Link Layer

The link layer, also known as the network interface layer, is the lowest layer of the TCP/IP model. It is responsible for the physical transmission of data. It includes the physical hardware and the protocols that are used to transmit data over a physical network. The link layer is equivalent to the physical and data link layers of the OSI model.

- **Key Functions:**
    - **Physical Transmission:** Transmits bits over a physical medium.
    - **Addressing:** Handles the physical addressing of devices.
    - **Error Detection:** Detects errors in the transmitted data.

### 2. Internet Layer

The internet layer is responsible for the logical transmission of data from the source to the destination. It is responsible for routing packets across the network. The internet layer is equivalent to the network layer of the OSI model.

- **Key Protocols:**
    - **Internet Protocol (IP):** The main protocol of the internet layer. It is responsible for addressing and routing packets.
    - **Address Resolution Protocol (ARP):** Used to resolve an IP address to a physical address.
    - **Internet Control Message Protocol (ICMP):** Used to send error messages and operational information.

### 3. Transport Layer

The transport layer is responsible for providing end-to-end communication between applications. It is responsible for ensuring that data is delivered reliably and in order. The transport layer is equivalent to the transport layer of the OSI model.

- **Key Protocols:**
    - **Transmission Control Protocol (TCP):** A connection-oriented protocol that provides reliable, ordered, and error-checked delivery of a stream of octets between applications.
    - **User Datagram Protocol (UDP):** A connectionless protocol that provides a simple, but unreliable, datagram service.

### 4. Application Layer

The application layer is the highest layer of the TCP/IP model. It is responsible for providing services to the user. The application layer is equivalent to the session, presentation, and application layers of the OSI model.

- **Key Protocols:**
    - **Hypertext Transfer Protocol (HTTP):** Used for transferring web pages.
    - **File Transfer Protocol (FTP):** Used for transferring files.
    - **Simple Mail Transfer Protocol (SMTP):** Used for sending email.
    - **Domain Name System (DNS):** Used for resolving domain names to IP addresses.

## Comparison of the OSI and TCP/IP Models

| Feature | OSI Model | TCP/IP Model |
|---|---|---|
| Number of Layers | 7 | 4 |
| Development | Developed by ISO | Developed by DARPA |
| Usage | A reference model | A practical model |
| Layers | Physical, Data Link, Network, Transport, Session, Presentation, Application | Link, Internet, Transport, Application |
| Reliability | Handled by the transport and data link layers | Handled by the transport layer |
| Addressing | Supports both connection-oriented and connectionless communication | Supports connectionless communication at the network layer |
