---
id: 06_The_Physical_Layer
aliases: []
tags: []
---


# 06. The Physical Layer

## Introduction to the Physical Layer

The physical layer is the first and lowest layer of the Open Systems Interconnection (OSI) model. It is responsible for the transmission and reception of unstructured raw data between a device and a physical transmission medium. It is the only layer that deals with the physical connectivity of two different stations. The physical layer contains information in the form of bits. It is responsible for transmitting individual bits from one node to the next.

When receiving data, this layer will get the signal received and convert it into 0s and 1s and send them to the Data Link layer, which will put the frame back together. When transmitting data, the physical layer will take the stream of bits from the Data Link layer and encode it into a signal that can be transmitted over the physical medium.

## Functions of the Physical Layer

- **Bit-by-Bit or Symbol-by-Symbol Delivery:** The physical layer is responsible for the movement of individual bits from one hop (node) to the next.
- **Line Configuration:** The physical layer is concerned with the connection of devices to the medium. In a point-to-point configuration, two devices are connected through a dedicated link. In a multipoint configuration, a link is shared among several devices.
- **Data Transmission:** The physical layer defines the transmission mode, whether it is simplex, half-duplex, or full-duplex. In simplex mode, only one device can send; the other can only receive. In half-duplex mode, two devices can send and receive, but not at the same time. In full-duplex mode, two devices can send and receive at the same time.
- **Topology:** The physical layer specifies the physical or logical arrangement of the network, such as a bus, a ring, or a star topology.
- **Signaling:** The physical layer determines the type of signaling to be used for transmitting the data. It can be either analog or digital.

## Transmission Media

Transmission media is the physical path between the transmitter and the receiver. It is the channel through which data is sent from one place to another. Transmission media can be broadly classified into two categories:

### Guided Media

Guided media, also known as wired or bounded transmission media, are physical media through which the signals are transmitted. They are called guided because they provide a physical path for the signal to travel.

- **Twisted-Pair Cable:** This is the most common type of guided media. It consists of two insulated copper wires twisted together. Twisted-pair cable is used in telephone lines and Ethernet networks.
- **Coaxial Cable:** Coaxial cable has a central copper core, an insulator, a braided metal shield, and an outer cover. It is used in cable television and some older Ethernet networks.
- **Fiber-Optic Cable:** Fiber-optic cable uses light to transmit data. It consists of a thin strand of glass or plastic that can carry light for long distances. Fiber-optic cable is used for high-speed data transmission.

### Unguided Media

Unguided media, also known as wireless or unbounded transmission media, do not have a physical path for the signal to travel. The signal is broadcast through the air.

- **Radio Waves:** Radio waves are used for a variety of wireless communication, including radio and television broadcasting, and Wi-Fi.
- **Microwaves:** Microwaves are used for long-distance communication, such as satellite communication and cellular networks.
- **Infrared:** Infrared is used for short-range communication, such as in remote controls.

## Digital and Analog Signals

### Digital Signals

A digital signal is a signal that is being used to represent data as a sequence of discrete values; at any given time it can only take on one of a finite number of values. This is in contrast to an analog signal, which represents continuous values; at any given time it represents a real number within a continuous range of values.

### Analog Signals

An analog signal is a continuous signal in which one time-varying quantity (such as voltage, pressure, etc.) represents another time-based variable. In other words, an analog signal is a representation of a signal that is continuous in both time and amplitude.

## Modulation and Demodulation

Modulation is the process of converting digital signals to analog signals. Demodulation is the process of converting analog signals back to digital signals. A modem is a device that performs both modulation and demodulation.
