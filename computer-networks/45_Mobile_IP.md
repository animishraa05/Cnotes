
# 45. Mobile IP

## Introduction to Mobile IP

Mobile IP is a communication protocol that allows a mobile device to move from one network to another while maintaining its permanent IP address. This is in contrast to the traditional IP routing, which is based on the assumption that a device's IP address is fixed. Mobile IP is an open standard defined by the Internet Engineering Task Force (IETF) in RFC 2002.

## How Mobile IP Works

Mobile IP works by using two IP addresses for a mobile device: a permanent IP address and a care-of address.

- **Permanent IP address:** The permanent IP address is the IP address that is assigned to the mobile device when it is on its home network.
- **Care-of address:** The care-of address is the IP address that is assigned to the mobile device when it is on a foreign network.

When a mobile device moves from its home network to a foreign network, it registers with a foreign agent on the foreign network. The foreign agent then sends a registration request to the home agent on the home network. The home agent then creates a tunnel to the foreign agent. All packets that are sent to the mobile device's permanent IP address are then forwarded through this tunnel to the foreign agent. The foreign agent then forwards the packets to the mobile device.

## Mobile IP Components

There are three main components of Mobile IP:

- **Mobile Node (MN):** The mobile node is the mobile device that is moving from one network to another.
- **Home Agent (HA):** The home agent is a router on the mobile node's home network. The home agent is responsible for forwarding packets to the mobile node when it is on a foreign network.
- **Foreign Agent (FA):** The foreign agent is a router on the foreign network that the mobile node is visiting. The foreign agent is responsible for forwarding packets from the home agent to the mobile node.

## Mobile IP vs. DHCP

Mobile IP and DHCP are two different protocols that are used to assign IP addresses to devices. The main difference between Mobile IP and DHCP is that Mobile IP allows a mobile device to maintain its permanent IP address while it is moving from one network to another, while DHCP does not.

## Advantages of Mobile IP

- **Mobility:** Mobile IP allows a mobile device to move from one network to another while maintaining its permanent IP address.
- **Transparency:** Mobile IP is transparent to the applications that are running on the mobile device.
- **Scalability:** Mobile IP is a scalable protocol that can be used to support a large number of mobile devices.

## Disadvantages of Mobile IP

- **Complexity:** Mobile IP can be complex to set up and manage.
- **Overhead:** Mobile IP adds overhead to the packets that are sent to the mobile device.
- **Security:** Mobile IP can be a security risk.
