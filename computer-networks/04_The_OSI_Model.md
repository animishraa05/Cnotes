---
id: 04_The_OSI_Model
aliases: []
tags: []
---

## 04. The OSI Model

## Introduction to the OSI Model

The Open Systems Interconnection (OSI) model is a conceptual framework that standardizes the functions of a telecommunication or computing system in terms of seven abstraction layers. The model was developed by the International Organization for Standardization (ISO) in 1984, and it is now considered the primary architectural model for intercomputer communications.

The OSI model divides the complex task of computer-to-computer communication into seven smaller, more manageable parts, called layers. Each layer is responsible for a specific part of the communication process. This layered approach allows for interoperability between different computer systems and network hardware.

## The Seven Layers of the OSI Model

### 1. Physical Layer

The physical layer is the lowest layer of the OSI model. It is responsible for the transmission and reception of unstructured raw data between a device and a physical transmission medium. It converts the digital bits into electrical, radio, or optical signals. 

- **Key Functions:**
    - Bit-by-bit or symbol-by-symbol delivery.
    - Defines the physical characteristics of the network, such as the type of cable, the type of connector, and the signaling.
    - Manages the way a device connects to the network medium.

### 2. Data Link Layer

The data link layer provides node-to-node data transfer—a link between two directly connected nodes. It detects and possibly corrects errors that may occur in the physical layer. It defines the protocol to establish and terminate a connection between two physically connected devices. It also defines the protocol for flow control between them.

- **Key Functions:**
    - **Framing:** Divides the stream of bits received from the network layer into manageable data units called frames.
    - **Physical Addressing:** Adds a header to the frame to define the sender and/or receiver of the frame.
    - **Error Control:** Detects and retransmits damaged or lost frames.
    - **Flow Control:** Prevents a fast sender from overwhelming a slow receiver.
    - **Access Control:** Determines which device has control over the link at any given time.

### 3. Network Layer

The network layer is responsible for packet forwarding including routing through intermediate routers. The network layer provides the functional and procedural means of transferring variable length data sequences from a source host on one network to a destination host on a different network. 

- **Key Functions:**
    - **Logical Addressing:** Adds a header to the packet which includes the logical addresses of the source and destination.
    - **Routing:** Determines the best path to move data from source to destination.

### 4. Transport Layer

The transport layer provides reliable transmission of data segments between points on a network, including segmentation, acknowledgement, and multiplexing. The transport layer provides the functional and procedural means of transferring variable-length data sequences from a source to a destination host via one or more networks, while maintaining the quality of service functions.

- **Key Functions:**
    - **Segmentation and Reassembly:** Segments the message into smaller chunks and reassembles them at the destination.
    - **Service-Point Addressing:** Ensures that the message is delivered to the correct process on the destination computer.
    - **Connection Control:** Can be either connectionless or connection-oriented.
    - **Flow Control:** End-to-end flow control.
    - **Error Control:** End-to-end error control.

### 5. Session Layer

The session layer controls the dialogues (connections) between computers. It establishes, manages and terminates the connections between the local and remote application. It provides for full-duplex, half-duplex, or simplex operation, and establishes checkpointing, adjournment, termination, and restart procedures.

- **Key Functions:**
    - **Dialog Control:** Allows two systems to enter into a dialog.
    - **Synchronization:** Allows a process to add checkpoints into a stream of data.

### 6. Presentation Layer

The presentation layer establishes a context between application-layer entities, in which the application-layer entities can use different syntax and semantics, as long as the presentation service provides a mapping between them. This layer is also responsible for data compression and encryption.

- **Key Functions:**
    - **Translation:** Translates data between different formats.
    - **Encryption:** Encrypts data for security.
    - **Compression:** Compresses data to reduce the amount of data to be transmitted.

### 7. Application Layer

The application layer is the OSI layer closest to the end user, which means that both the OSI application layer and the user interact directly with the software application. This layer interacts with software applications that implement a communicating component. Such application programs fall outside the scope of the OSI model.

- **Key Functions:**
    - **Network Virtual Terminal:** Allows a user to log on to a remote host.
    - **File Transfer, Access, and Management (FTAM):** Allows a user to access files in a remote host, to retrieve files from a remote host, and to manage or control files in a remote host.
    - **Mail Services:** Provides the basis for e-mail forwarding and storage.
    - [ ] **Directory Services:** Provides distributed database sources and access for global information about various objects and services.
