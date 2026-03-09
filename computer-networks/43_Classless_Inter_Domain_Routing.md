
# 43. Classless Inter-Domain Routing (CIDR)

## Introduction to CIDR

Classless Inter-Domain Routing (CIDR) is a method for allocating IP addresses and for IP routing. The Internet Engineering Task Force introduced CIDR in 1993 to replace the previous classful network addressing architecture on the Internet. Its goal was to slow the growth of routing tables on routers across the Internet, and to help slow the rapid exhaustion of IPv4 addresses.

## How CIDR Works

CIDR works by using a variable-length subnet mask (VLSM). A VLSM allows a network to be divided into subnets of different sizes. This is in contrast to the previous classful network addressing architecture, which used a fixed-length subnet mask.

CIDR also uses a new notation for writing IP addresses. The new notation is called CIDR notation. CIDR notation is a more compact way of writing IP addresses and subnet masks. For example, the IP address 192.168.1.0 with the subnet mask 255.255.255.0 can be written in CIDR notation as 192.168.1.0/24.

## CIDR Example

Suppose you have a network with the IP address 192.168.1.0/24. You want to create 4 subnets. To do this, you will need to borrow 2 bits from the host portion of the IP address. The new subnet mask will be /26.

The following table shows the 4 subnets that you can create:

| Subnet | Network Address | Broadcast Address | Host Range |
|---|---|---|---|
| 1 | 192.168.1.0/26 | 192.168.1.63 | 192.168.1.1 - 192.168.1.62 |
| 2 | 192.168.1.64/26 | 192.168.1.127 | 192.168.1.65 - 192.168.1.126 |
| 3 | 192.168.1.128/26 | 192.168.1.191 | 192.168.1.129 - 192.168.1.190 |
| 4 | 192.168.1.192/26 | 192.168.1.255 | 192.168.1.193 - 192.168.1.254 |

## Advantages of CIDR

- **Slows the exhaustion of IPv4 addresses:** CIDR can slow the exhaustion of IPv4 addresses by allowing a network to be divided into subnets of different sizes.
- **Reduces the size of routing tables:** CIDR can reduce the size of routing tables by allowing multiple routes to be aggregated into a single route.
- **More flexible:** CIDR is more flexible than the previous classful network addressing architecture.

## Disadvantages of CIDR

- **Complexity:** CIDR can be complex to set up and manage.
