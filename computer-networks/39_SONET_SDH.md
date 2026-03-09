
# 39. Synchronous Optical Networking (SONET) and Synchronous Digital Hierarchy (SDH)

## Introduction to SONET/SDH

Synchronous Optical Networking (SONET) and Synchronous Digital Hierarchy (SDH) are standardized protocols that transfer multiple digital bit streams synchronously over optical fiber using lasers or highly coherent light from light-emitting diodes (LEDs). At low transmission rates data can also be transferred via an electrical interface.

SONET is the United States version of the standard, and SDH is the international version of the standard. The two standards are very similar, and they are often used interchangeably.

## SONET/SDH Hierarchy

SONET/SDH defines a hierarchy of data rates. The base rate is 51.84 Mbps. The other rates are multiples of the base rate.

| SONET Level | SDH Level | Data Rate (Mbps) |
|---|---|---|
| OC-1 | STM-1 | 51.84 |
| OC-3 | STM-1 | 155.52 |
| OC-12 | STM-4 | 622.08 |
| OC-48 | STM-16 | 2488.32 |
| OC-192 | STM-64 | 9953.28 |
| OC-768 | STM-256 | 39813.12 |

## SONET/SDH Frame

A SONET/SDH frame has a size of 810 bytes. The frame is divided into two parts: the transport overhead and the synchronous payload envelope (SPE).

The transport overhead contains information about the frame, such as the frame synchronization, the section trace, and the section error monitoring.

The SPE contains the data that is being transmitted.

## SONET/SDH vs. ATM

SONET/SDH and ATM are two different networking technologies. The main difference between SONET/SDH and ATM is the way that they transmit data. SONET/SDH uses a synchronous time-division multiplexing (TDM) scheme, while ATM uses an asynchronous cell-based scheme.

SONET/SDH is a connection-oriented protocol, which means that a connection must be established before data can be transmitted. ATM is also a connection-oriented protocol.

SONET/SDH is more efficient than ATM for carrying constant bit rate (CBR) traffic, such as voice and video. ATM is more efficient than SONET/SDH for carrying variable bit rate (VBR) traffic, such as data.

## Decline of SONET/SDH

SONET/SDH was once a popular networking technology, but it has been largely replaced by other technologies, such as Ethernet and MPLS. The decline of SONET/SDH is due to a number of factors, including the high cost of SONET/SDH hardware, the complexity of the SONET/SDH protocol, and the rise of Ethernet and MPLS.
