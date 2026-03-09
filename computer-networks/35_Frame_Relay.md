
# 35. Frame Relay

## Introduction to Frame Relay

Frame Relay is a standardized wide area network (WAN) technology that specifies the physical and data link layers of digital telecommunications channels using a packet switching methodology. Originally designed for transport across Integrated Services Digital Network (ISDN) infrastructure, it may be used today in a variety of other network interfaces. Frame Relay is a connection-oriented protocol that uses variable-size frames to transmit data.

## Frame Relay Standards

Frame Relay is standardized by the International Telecommunication Union (ITU) and the American National Standards Institute (ANSI). The Frame Relay standards define the physical layer and the data link layer of the protocol.

## Frame Relay Frame

A Frame Relay frame has the following format:

- **Flag:** The flag is a 1-byte field that indicates the start and end of the frame.
- **Address:** The address is a 2-byte field that contains the data link connection identifier (DLCI).
- **Data:** The data field is a variable-length field that contains the data that is being transmitted.
- **Frame Check Sequence (FCS):** The FCS is a 2-byte field that contains a checksum of the frame. The FCS is used to detect errors in the frame.

## Data Link Connection Identifier (DLCI)

A DLCI is a 10-bit number that identifies a virtual circuit. A virtual circuit is a logical connection between two devices on a Frame Relay network. A virtual circuit can be either a permanent virtual circuit (PVC) or a switched virtual circuit (SVC).

- **Permanent Virtual Circuit (PVC):** A PVC is a permanent connection between two devices. A PVC is established by the network administrator and remains in place until it is manually removed.
- **Switched Virtual Circuit (SVC):** An SVC is a temporary connection between two devices. An SVC is established on demand and is torn down when it is no longer needed.

## Frame Relay vs. X.25

Frame Relay and X.25 are two different WAN technologies. The main difference between Frame Relay and X.25 is the way that they handle errors. Frame Relay does not perform error correction, while X.25 does. This makes Frame Relay more efficient than X.25, but it also makes it less reliable.

## Decline of Frame Relay

Frame Relay was once a popular WAN technology, but it has been largely replaced by other technologies, such as MPLS and Ethernet. The decline of Frame Relay is due to a number of factors, including the high cost of Frame Relay hardware, the complexity of the Frame Relay protocol, and the rise of MPLS and Ethernet.
