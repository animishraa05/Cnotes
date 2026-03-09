
# 31. Ethernet

## Introduction to Ethernet

Ethernet is a family of computer networking technologies commonly used in local area networks (LAN), metropolitan area networks (MAN) and wide area networks (WAN). It was commercially introduced in 1980 and first standardized in 1983 as IEEE 802.3. Ethernet has since been refined to support higher bit rates, a greater number of nodes, and longer link distances, but retains much backward compatibility. Over time, Ethernet has largely replaced competing wired LAN technologies such as Token Ring, FDDI and ARCNET.

## Ethernet Standards

There are many different Ethernet standards, but some of the most common standards are:

- **10BASE-T:** 10BASE-T is an Ethernet standard that uses twisted-pair cable and has a data rate of 10 Mbps.
- **100BASE-TX:** 100BASE-TX is an Ethernet standard that uses twisted-pair cable and has a data rate of 100 Mbps.
- **1000BASE-T:** 1000BASE-T is an Ethernet standard that uses twisted-pair cable and has a data rate of 1 Gbps.
- **10GBASE-T:** 10GBASE-T is an Ethernet standard that uses twisted-pair cable and has a data rate of 10 Gbps.

## Ethernet Frame

An Ethernet frame is a data packet that is transmitted over an Ethernet network. An Ethernet frame has the following format:

- **Preamble:** The preamble is a 7-byte field that is used to synchronize the receiver with the sender.
- **Start Frame Delimiter (SFD):** The SFD is a 1-byte field that indicates the start of the frame.
- **Destination MAC Address:** The destination MAC address is a 6-byte field that contains the MAC address of the destination device.
- **Source MAC Address:** The source MAC address is a 6-byte field that contains the MAC address of the source device.
- **Length/Type:** The length/type field is a 2-byte field that indicates the length of the data field or the type of the protocol.
- **Data:** The data field is a variable-length field that contains the data that is being transmitted.
- **Frame Check Sequence (FCS):** The FCS is a 4-byte field that contains a checksum of the frame. The FCS is used to detect errors in the frame.

## Carrier Sense Multiple Access with Collision Detection (CSMA/CD)

CSMA/CD is a media access control method used by Ethernet networks. CSMA/CD works as follows:

1. A device that wants to transmit data first listens to the network to see if it is busy.
2. If the network is not busy, the device starts to transmit data.
3. While the device is transmitting data, it continues to listen to the network to see if a collision has occurred.
4. If a collision occurs, the device stops transmitting data and sends a jam signal.
5. The device then waits a random amount of time before trying to transmit data again.

## Ethernet Switches

An Ethernet switch is a network device that connects multiple Ethernet devices together. An Ethernet switch works by learning the MAC addresses of the devices that are connected to it. When a frame is received, the switch looks at the destination MAC address and forwards the frame to the port that is connected to the destination device. This is in contrast to an Ethernet hub, which forwards the frame to all of the ports.
