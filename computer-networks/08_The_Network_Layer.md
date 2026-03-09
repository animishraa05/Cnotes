
# 08. The Network Layer

## Introduction to the Network Layer

The network layer is the third layer of the OSI model. It is responsible for providing logical addressing and routing of data from the source to the destination. The network layer is responsible for the host-to-host delivery of packets. It is the layer that is responsible for the forwarding of data packets across different networks.

## Functions of the Network Layer

- **Logical Addressing:** The network layer is responsible for adding a header to the packet that contains the logical address of the source and destination. The logical address is a unique address that is assigned to each device on the network.
- **Routing:** The network layer is responsible for determining the best path for the data to travel from the source to the destination. This is done by using routing algorithms.
- **Packetizing:** The network layer is responsible for encapsulating the data from the upper layers into packets. A packet is a variable-length data unit that is used by the network layer.
- **Fragmentation:** The network layer is responsible for breaking up large packets into smaller packets so that they can be transmitted over the network.

## Key Protocols of the Network Layer

- **Internet Protocol (IP):** The Internet Protocol is the main protocol of the network layer. It is responsible for addressing and routing packets. There are two versions of IP: IPv4 and IPv6.
- **Address Resolution Protocol (ARP):** The Address Resolution Protocol is used to resolve an IP address to a physical address. The physical address is the MAC address of the device.
- **Internet Control Message Protocol (ICMP):** The Internet Control Message Protocol is used to send error messages and operational information. For example, the ping command uses ICMP to test the connectivity between two devices.
- **Internet Group Management Protocol (IGMP):** The Internet Group Management Protocol is used to manage multicast groups. A multicast group is a group of devices that receive the same data.

## Routing Algorithms

Routing algorithms are used to determine the best path for the data to travel from the source to the destination. There are two main types of routing algorithms:

- **Distance Vector Routing:** In distance vector routing, each router maintains a table that contains the distance to every other router in the network. The distance is usually measured in terms of the number of hops.
- **Link State Routing:** In link state routing, each router maintains a map of the entire network. The map contains information about the state of each link in the network. This allows each router to calculate the best path to every other router in the network.

## IP Addressing

An IP address is a unique address that is assigned to each device on a network. There are two versions of IP addressing: IPv4 and IPv6.

### IPv4

An IPv4 address is a 32-bit address that is divided into four octets. Each octet is a number between 0 and 255. An IPv4 address is usually written in dotted-decimal notation, for example, 192.168.1.1.

### IPv6

An IPv6 address is a 128-bit address that is divided into eight 16-bit blocks. Each block is written as four hexadecimal digits. An IPv6 address is usually written in hexadecimal notation, for example, 2001:0db8:85a3:0000:0000:8a2e:0370:7334.
