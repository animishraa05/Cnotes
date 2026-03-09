
# 07. The Data Link Layer

## Introduction to the Data Link Layer

The data link layer is the second layer of the OSI model. It is responsible for providing reliable, error-free transmission of data between two nodes on the same local network. The data link layer takes the packets from the network layer and encapsulates them into frames for transmission. It is also responsible for flow control, which ensures that a fast sender does not overwhelm a slow receiver.

## Functions of the Data Link Layer

- **Framing:** The data link layer divides the stream of bits received from the network layer into manageable data units called frames.
- **Physical Addressing:** The data link layer adds a header to the frame to define the physical address of the source and destination nodes on the local network.
- **Error Control:** The data link layer is responsible for detecting and correcting errors that occur in the physical layer. It adds a trailer to the frame to detect errors.
- **Flow Control:** The data link layer is responsible for regulating the flow of data so that a fast sender does not overwhelm a slow receiver.
- **Access Control:** When two or more devices are connected to the same link, the data link layer is responsible for determining which device has control over the link at any given time.

## Sublayers of the Data Link Layer

The data link layer is often divided into two sublayers:

- **Logical Link Control (LLC) Sublayer:** The LLC sublayer is responsible for identifying and encapsulating network layer protocols, and for controlling error checking and frame synchronization.
- **Media Access Control (MAC) Sublayer:** The MAC sublayer is responsible for controlling how devices in a network gain access to the medium and permission to transmit data.

## Error Detection and Correction

### Error Detection

Error detection is the process of detecting errors that occur during the transmission of data. There are several methods for error detection, including:

- **Parity Check:** In this method, a parity bit is added to each character. The parity bit is set to 1 if the number of 1s in the character is odd, and to 0 if the number of 1s is even.
- **Checksum:** In this method, a checksum is calculated for the data and appended to the data. The receiver recalculates the checksum and compares it to the received checksum. If the two checksums are the same, then no error has occurred.
- **Cyclic Redundancy Check (CRC):** This is a more powerful error detection method than the parity check or the checksum. In this method, a CRC is calculated for the data and appended to the data. The receiver recalculates the CRC and compares it to the received CRC. If the two CRCs are the same, then no error has occurred.

### Error Correction

Error correction is the process of correcting errors that occur during the transmission of data. There are two main methods for error correction:

- **Forward Error Correction (FEC):** In this method, the sender adds redundant data to the message. The receiver can use this redundant data to detect and correct errors.
- **Automatic Repeat Request (ARQ):** In this method, the receiver detects an error and requests the sender to retransmit the data.

## Flow Control

Flow control is the process of managing the rate of data transmission between two nodes to prevent a fast sender from overwhelming a slow receiver. There are two main methods for flow control:

- **Stop-and-Wait:** In this method, the sender sends one frame and then waits for an acknowledgment from the receiver before sending the next frame.
- **Sliding Window:** In this method, the sender can send multiple frames before waiting for an acknowledgment. The receiver can acknowledge multiple frames with a single acknowledgment.
