
# 42. Subnetting

## Introduction to Subnetting

Subnetting is the process of dividing a single, large IP network into smaller, more manageable subnetworks, or subnets. This is done by borrowing bits from the host portion of an IP address and using them to create a subnet mask. The subnet mask is then used to determine which subnet a particular IP address belongs to.

## How Subnetting Works

To subnet a network, you first need to decide how many subnets you need and how many hosts you need per subnet. Once you have this information, you can then determine the subnet mask that you will use.

The subnet mask is a 32-bit number that is used to divide the IP address into a network portion and a host portion. The network portion of the IP address identifies the subnet, and the host portion of the IP address identifies the host on the subnet.

To calculate the subnet mask, you first need to determine the number of bits that you will borrow from the host portion of the IP address. The number of bits that you borrow will determine the number of subnets that you can create. The more bits that you borrow, the more subnets that you can create.

Once you have determined the number of bits that you will borrow, you can then calculate the subnet mask. The subnet mask is calculated by setting the bits that you have borrowed to 1 and the remaining bits to 0.

## Subnetting Example

Suppose you have a class C network with the IP address 192.168.1.0. You want to create 4 subnets. To do this, you will need to borrow 2 bits from the host portion of the IP address. The subnet mask will be 255.255.255.192.

The following table shows the 4 subnets that you can create:

| Subnet | Network Address | Broadcast Address | Host Range |
|---|---|---|---|
| 1 | 192.168.1.0 | 192.168.1.63 | 192.168.1.1 - 192.168.1.62 |
| 2 | 192.168.1.64 | 192.168.1.127 | 192.168.1.65 - 192.168.1.126 |
| 3 | 192.168.1.128 | 192.168.1.191 | 192.168.1.129 - 192.168.1.190 |
| 4 | 192.168.1.192 | 192.168.1.255 | 192.168.1.193 - 192.168.1.254 |

## Advantages of Subnetting

- **Improved performance:** Subnetting can improve the performance of a network by reducing the amount of broadcast traffic.
- **Increased security:** Subnetting can increase security by isolating the different subnets from each other.
- **Simplified administration:** Subnetting can simplify the administration of a network by making it easier to manage the different subnets.

## Disadvantages of Subnetting

- **Complexity:** Subnetting can be complex to set up and manage.
- **Wasted IP addresses:** Subnetting can waste IP addresses.
