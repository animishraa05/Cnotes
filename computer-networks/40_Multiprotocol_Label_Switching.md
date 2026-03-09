
# 40. Multiprotocol Label Switching (MPLS)

## Introduction to MPLS

Multiprotocol Label Switching (MPLS) is a routing technique in telecommunications networks that directs data from one node to the next based on short path labels rather than long network addresses, avoiding complex lookups in a routing table. The labels identify virtual links (paths) between distant nodes rather than endpoints. MPLS can encapsulate packets of various network protocols, hence the "multiprotocol" reference in its name. MPLS supports a range of access technologies, including T1/E1, ATM, Frame Relay, and DSL.

## How MPLS Works

MPLS works by adding a label to each packet. The label is a short, fixed-length value that is used to identify the packet's path through the network. The routers in the network use the label to forward the packet to the next hop.

The first router in the network, called the ingress router, adds the label to the packet. The last router in the network, called the egress router, removes the label from the packet.

The routers in between the ingress and egress routers, called the label switch routers (LSRs), use the label to forward the packet to the next hop.

## MPLS Labels

An MPLS label is a 32-bit value that is divided into four parts:

- **Label:** The label is a 20-bit value that is used to identify the packet's path through the network.
- **Experimental (EXP):** The EXP is a 3-bit value that is used for quality of service (QoS).
- **Bottom of Stack (S):** The S is a 1-bit value that indicates whether the label is the last label in the stack.
- **Time to Live (TTL):** The TTL is an 8-bit value that is used to prevent packets from looping in the network.

## MPLS vs. IP Routing

MPLS and IP routing are two different routing techniques. The main difference between MPLS and IP routing is the way that they forward packets. MPLS forwards packets based on short path labels, while IP routing forwards packets based on long network addresses.

MPLS is more efficient than IP routing, because it does not need to perform complex lookups in a routing table. MPLS is also more scalable than IP routing, because it can support a larger number of nodes.

## Advantages of MPLS

- **Performance:** MPLS can improve the performance of a network by reducing the amount of time that it takes to forward packets.
- **Scalability:** MPLS is more scalable than IP routing, because it can support a larger number of nodes.
- **Quality of Service (QoS):** MPLS can be used to provide QoS for different types of traffic.
- **Traffic Engineering:** MPLS can be used to engineer the traffic on a network.
- **Virtual Private Networks (VPNs):** MPLS can be used to create VPNs.

## Disadvantages of MPLS

- **Complexity:** MPLS can be complex to set up and manage.
- **Cost:** MPLS can be expensive to implement.
