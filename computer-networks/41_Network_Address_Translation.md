
# 41. Network Address Translation (NAT)

## Introduction to NAT

Network Address Translation (NAT) is a method of remapping one IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device. The term is most commonly used to refer to the process where a private IP address is translated into a public IP address. This is done to conserve the limited number of public IPv4 addresses.

## How NAT Works

NAT works by creating a table that maps private IP addresses to public IP addresses. When a packet from a private IP address is sent to a public IP address, the NAT device looks up the private IP address in the table and replaces it with the corresponding public IP address. The NAT device also creates a new port number for the packet. The new port number is used to distinguish the packet from other packets that are being sent from the same public IP address.

When a packet from a public IP address is sent to a private IP address, the NAT device looks up the public IP address and port number in the table and replaces them with the corresponding private IP address and port number.

## Types of NAT

There are three main types of NAT:

- **Static NAT:** Static NAT maps a single private IP address to a single public IP address. Static NAT is often used to allow a device on a private network to be accessible from the internet.
- **Dynamic NAT:** Dynamic NAT maps a pool of private IP addresses to a pool of public IP addresses. Dynamic NAT is often used to allow multiple devices on a private network to share a single public IP address.
- **Port Address Translation (PAT):** PAT is a type of dynamic NAT that maps multiple private IP addresses to a single public IP address by using different port numbers. PAT is the most common type of NAT.

## Advantages of NAT

- **Conserves public IPv4 addresses:** NAT can be used to conserve the limited number of public IPv4 addresses.
- **Increases security:** NAT can increase security by hiding the private IP addresses of the devices on a private network.
- **Flexibility:** NAT can be used to create a more flexible network.

## Disadvantages of NAT

- **Complexity:** NAT can be complex to set up and manage.
- **Breaks end-to-end connectivity:** NAT breaks the end-to-end connectivity model of the internet.
- **Can interfere with some applications:** NAT can interfere with some applications, such as peer-to-peer applications.
