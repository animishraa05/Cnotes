
# 33. Fiber Distributed Data Interface (FDDI)

## Introduction to FDDI

Fiber Distributed Data Interface (FDDI) is a standard for data transmission in a local area network. It uses a dual-ring topology and a token-passing scheme to control access to the medium. FDDI was developed in the 1980s as a high-speed alternative to Ethernet and Token Ring. It has a data rate of 100 Mbps.

## FDDI Standards

FDDI is standardized by the American National Standards Institute (ANSI) and the International Organization for Standardization (ISO). The FDDI standards define the physical layer and the media access control (MAC) layer of the protocol.

## FDDI Topology

FDDI uses a dual-ring topology. The dual-ring topology consists of two independent rings that transmit data in opposite directions. The primary ring is used for data transmission, and the secondary ring is used for backup. If the primary ring fails, the secondary ring can be used to continue data transmission.

## FDDI Frame

An FDDI frame has the following format:

- **Preamble:** The preamble is a 1-byte field that is used to synchronize the receiver with the sender.
- **Start Delimiter (SD):** The SD is a 1-byte field that indicates the start of the frame.
- **Frame Control (FC):** The FC is a 1-byte field that indicates the type of the frame.
- **Destination MAC Address:** The destination MAC address is a 6-byte field that contains the MAC address of the destination station.
- **Source MAC Address:** The source MAC address is a 6-byte field that contains the MAC address of the source station.
- **Data:** The data field is a variable-length field that contains the data that is being transmitted.
- **Frame Check Sequence (FCS):** The FCS is a 4-byte field that contains a checksum of the frame. The FCS is used to detect errors in the frame.
- **End Delimiter (ED):** The ED is a 1-byte field that indicates the end of the frame.
- **Frame Status (FS):** The FS is a 3-byte field that contains the error detected bit, the address recognized bit, and the frame copied bit.

## FDDI vs. Ethernet

FDDI and Ethernet are two different LAN technologies. The main difference between FDDI and Ethernet is the way that they control access to the medium. FDDI uses a token-passing scheme, while Ethernet uses a CSMA/CD scheme.

FDDI is a deterministic protocol, which means that each station is guaranteed to be able to transmit data within a certain amount of time. Ethernet is a probabilistic protocol, which means that there is no guarantee that a station will be able to transmit data within a certain amount of time.

FDDI is more reliable than Ethernet, but it is also more expensive. Ethernet is less reliable than FDDI, but it is also less expensive.

## Decline of FDDI

FDDI was once a popular LAN technology, but it has been largely replaced by Ethernet. The decline of FDDI is due to a number of factors, including the high cost of FDDI hardware, the complexity of the FDDI protocol, and the rise of Fast Ethernet and Gigabit Ethernet.
