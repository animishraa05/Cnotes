
# 18. Firewalls

## Introduction to Firewalls

A firewall is a network security system that monitors and controls incoming and outgoing network traffic based on predetermined security rules. A firewall typically establishes a barrier between a trusted internal network and an untrusted external network, such as the Internet. Firewalls are a first line of defense in network security.

## How Firewalls Work

A firewall works by inspecting the packets that are sent and received on a network. The firewall can be configured to allow or deny packets based on a variety of criteria, such as the source IP address, the destination IP address, the port number, and the protocol. 

## Types of Firewalls

There are many different types of firewalls, but some of the most common types are:

- **Packet-Filtering Firewalls:** A packet-filtering firewall is the most basic type of firewall. It works by inspecting the header of each packet and deciding whether to allow or deny the packet based on a set of rules. Packet-filtering firewalls are fast and efficient, but they are not very secure.
- **Stateful Inspection Firewalls:** A stateful inspection firewall is more secure than a packet-filtering firewall. It works by keeping track of the state of each connection. This allows the firewall to make more intelligent decisions about whether to allow or deny a packet. Stateful inspection firewalls are more secure than packet-filtering firewalls, but they are also more complex and expensive.
- **Proxy Firewalls:** A proxy firewall is the most secure type of firewall. It works by acting as a proxy for all traffic between the internal network and the external network. This means that all traffic must pass through the proxy firewall, which can inspect the traffic for malicious content. Proxy firewalls are the most secure type of firewall, but they are also the most complex and expensive.
- **Next-Generation Firewalls (NGFW):** A next-generation firewall is a type of firewall that combines the features of a traditional firewall with other security features, such as an intrusion prevention system (IPS), an antivirus, and a web filter. NGFWs are designed to provide a more comprehensive security solution than traditional firewalls.

## Firewall Architectures

There are many different firewall architectures, but some of the most common architectures are:

- **Screened Host Firewall:** A screened host firewall is a firewall that is placed between the internal network and the external network. The firewall is configured to allow only certain types of traffic to pass through.
- **Screened Subnet Firewall:** A screened subnet firewall is a firewall that is placed between two networks. The firewall is configured to allow only certain types of traffic to pass through.
- **Dual-Homed Host Firewall:** A dual-homed host firewall is a firewall that has two network interfaces. One network interface is connected to the internal network, and the other network interface is connected to the external network. The firewall is configured to allow only certain types of traffic to pass through.

## Firewall Rules

Firewall rules are used to control the traffic that is allowed to pass through the firewall. Firewall rules can be based on a variety of criteria, such as the source IP address, the destination IP address, the port number, and the protocol.

Firewall rules are typically processed in order. This means that the first rule that matches the traffic is the rule that is applied. If no rule matches the traffic, then the default rule is applied. The default rule is typically to deny all traffic.
