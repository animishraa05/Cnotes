
# 32. Token Ring

## Introduction to Token Ring

Token Ring is a local area network (LAN) technology in which all stations are connected in a ring and a token-passing scheme is used to control access to the medium. The token is a special bit pattern that circulates around the ring. A station that wants to transmit data must wait until it receives the token. The station then changes the token to a frame, appends the data to the frame, and sends the frame to the next station in the ring. The frame circulates around the ring until it reaches the destination station. The destination station copies the data from the frame and then sends the frame back to the source station. The source station then removes the data from the frame and changes the frame back to a token.

## Token Ring Standards

Token Ring was originally developed by IBM in the 1980s. It was later standardized by the IEEE as IEEE 802.5. The IEEE 802.5 standard defines two types of Token Ring: 4 Mbps and 16 Mbps.

## Token Ring Frame

A Token Ring frame has the following format:

- **Start Delimiter (SD):** The SD is a 1-byte field that indicates the start of the frame.
- **Access Control (AC):** The AC is a 1-byte field that contains the token bit, the monitor bit, and the priority bits.
- **Frame Control (FC):** The FC is a 1-byte field that indicates the type of the frame.
- **Destination MAC Address:** The destination MAC address is a 6-byte field that contains the MAC address of the destination station.
- **Source MAC Address:** The source MAC address is a 6-byte field that contains the MAC address of the source station.
- **Data:** The data field is a variable-length field that contains the data that is being transmitted.
- **Frame Check Sequence (FCS):** The FCS is a 4-byte field that contains a checksum of the frame. The FCS is used to detect errors in the frame.
- **End Delimiter (ED):** The ED is a 1-byte field that indicates the end of the frame.
- **Frame Status (FS):** The FS is a 1-byte field that contains the address recognized bit and the frame copied bit.

## Token Ring vs. Ethernet

Token Ring and Ethernet are two different LAN technologies. The main difference between Token Ring and Ethernet is the way that they control access to the medium. Token Ring uses a token-passing scheme, while Ethernet uses a CSMA/CD scheme.

Token Ring is a deterministic protocol, which means that each station is guaranteed to be able to transmit data within a certain amount of time. Ethernet is a probabilistic protocol, which means that there is no guarantee that a station will be able to transmit data within a certain amount of time.

Token Ring is more reliable than Ethernet, but it is also more expensive. Ethernet is less reliable than Token Ring, but it is also less expensive.

## Decline of Token Ring

Token Ring was once a popular LAN technology, but it has been largely replaced by Ethernet. The decline of Token Ring is due to a number of factors, including the high cost of Token Ring hardware, the complexity of the Token Ring protocol, and the rise of switched Ethernet.
