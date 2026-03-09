
# 34. Asynchronous Transfer Mode (ATM)

## Introduction to ATM

Asynchronous Transfer Mode (ATM) is a switching technique for telecommunication networks. It is a connection-oriented protocol that uses fixed-size cells to transmit data. ATM was developed in the 1980s as a high-speed alternative to traditional circuit-switched and packet-switched networks. It has a data rate of up to 10 Gbps.

## ATM Standards

ATM is standardized by the International Telecommunication Union (ITU) and the ATM Forum. The ATM standards define the physical layer, the ATM layer, and the ATM adaptation layer (AAL) of the protocol.

## ATM Cell

An ATM cell is a fixed-size packet that is used to transmit data in an ATM network. An ATM cell has a size of 53 bytes. The first 5 bytes of the cell are the header, and the remaining 48 bytes are the payload.

The header of an ATM cell contains the following fields:

- **Generic Flow Control (GFC):** The GFC is a 4-bit field that is used to control the flow of traffic.
- **Virtual Path Identifier (VPI):** The VPI is an 8-bit field that identifies the virtual path.
- **Virtual Channel Identifier (VCI):** The VCI is a 16-bit field that identifies the virtual channel.
- **Payload Type (PT):** The PT is a 3-bit field that indicates the type of the payload.
- **Cell Loss Priority (CLP):** The CLP is a 1-bit field that indicates the priority of the cell.
- **Header Error Control (HEC):** The HEC is an 8-bit field that is used to detect errors in the header.

## ATM Adaptation Layer (AAL)

The ATM adaptation layer (AAL) is responsible for adapting the data from the higher layers to the ATM layer. The AAL is divided into two sublayers: the convergence sublayer (CS) and the segmentation and reassembly (SAR) sublayer.

The CS is responsible for providing a specific service to the higher layers. The SAR is responsible for segmenting the data from the higher layers into ATM cells and for reassembling the ATM cells into data at the destination.

There are four types of AALs:

- **AAL1:** AAL1 is used for constant bit rate (CBR) services, such as voice and video.
- **AAL2:** AAL2 is used for variable bit rate (VBR) services, such as compressed voice and video.
- **AAL3/4:** AAL3/4 is used for variable bit rate (VBR) services, such as data.
- **AAL5:** AAL5 is used for variable bit rate (VBR) services, such as data.

## ATM vs. Ethernet

ATM and Ethernet are two different networking technologies. The main difference between ATM and Ethernet is the way that they transmit data. ATM uses fixed-size cells, while Ethernet uses variable-size frames.

ATM is a connection-oriented protocol, which means that a connection must be established before data can be transmitted. Ethernet is a connectionless protocol, which means that data can be transmitted without establishing a connection.

ATM is more reliable than Ethernet, but it is also more expensive. Ethernet is less reliable than ATM, but it is also less expensive.

## Decline of ATM

ATM was once a popular networking technology, but it has been largely replaced by Ethernet. The decline of ATM is due to a number of factors, including the high cost of ATM hardware, the complexity of the ATM protocol, and the rise of Fast Ethernet and Gigabit Ethernet.
