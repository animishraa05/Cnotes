
# 02. Network Topologies

## What is a Network Topology?

Network topology refers to the arrangement of the various elements (links, nodes, etc.) of a computer network. Essentially, it is the topological structure of a network and may be depicted physically or logically. Physical topology is the placement of the various components of a network, including device location and cable installation, while logical topology illustrates how data flows within a network, regardless of its physical design. 

## Physical Topologies

### Bus Topology

In a bus topology, all devices are connected to a single cable, which is called the bus or backbone. This is a simple and inexpensive topology to implement. However, if the main cable fails, the entire network will go down.

- **Advantages:**
    - Easy to install and understand.
    - Requires less cable than other topologies.
    - Inexpensive.

- **Disadvantages:**
    - Difficult to troubleshoot.
    - The entire network shuts down if there is a break in the main cable.
    - Performance degrades as more devices are added.

### Ring Topology

In a ring topology, every device has exactly two neighbors for communication purposes. All messages travel through a ring in the same direction (either "clockwise" or "counterclockwise"). A failure in any cable or device breaks the loop and can take down the entire network.

- **Advantages:**
    - All data flows in one direction, reducing the chance of packet collisions.
    - A network server is not needed to control network connectivity between each workstation.
    - Data can be transferred at high speeds.

- **Disadvantages:**
    - The entire network will be impacted if one workstation shuts down.
    - The hardware needed to connect each workstation to the network is more expensive than Ethernet cards and hubs/switches.

### Star Topology

In a star topology, all devices are connected to a central hub or switch. This is the most common topology used in home and office networks.

- **Advantages:**
    - Easy to install and manage.
    - If one node or cable fails, the other nodes will not be affected.
    - Easy to add new nodes.

- **Disadvantages:**
    - If the central hub or switch fails, the entire network will go down.
    - Requires more cable than a bus topology.

### Tree Topology

A tree topology is a hybrid topology that combines the characteristics of bus and star topologies. It consists of groups of star-configured workstations connected to a linear bus backbone cable.

- **Advantages:**
    - Point-to-point wiring for individual segments.
    - Supported by several hardware and software venders.
    - Easy to extend.

- **Disadvantages:**
    - Overall length of each segment is limited by the type of cabling used.
    - If the backbone line breaks, the entire segment goes down.
    - More difficult to configure and wire than other topologies.

### Mesh Topology

In a mesh topology, every device is connected to every other device. This provides for a high level of redundancy, but it is also expensive to implement.

- **Advantages:**
    - Can handle high amounts of traffic.
    - If one connection fails, the other connections can still be used.
    - Easy to troubleshoot.

- **Disadvantages:**
    - Expensive to implement.
    - Difficult to manage.

### Hybrid Topology

A hybrid topology is a combination of two or more different topologies. For example, a tree topology is a hybrid of bus and star topologies. A hybrid topology is often used in large networks.

- **Advantages:**
    - Flexible and reliable.
    - Can be designed according to the requirements of the organization.

- **Disadvantages:**
    - Complex to design.
    - Expensive.

## Logical Topologies

Logical topology, in contrast to physical topology, is the way that the signals act on the network media, or the way that the data passes through the network from one device to the next without regard to the physical interconnection of the devices. A network's logical topology is not necessarily the same as its physical topology.

For example, twisted pair Ethernet is a logical bus topology in a physical star topology layout. While IBM's Token Ring is a logical ring topology, it is physically set up in a star topology.
