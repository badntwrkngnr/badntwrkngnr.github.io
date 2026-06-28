---
title: "Networking Fundamentals"
slug: "networking-fundamentals" 
date: 2025-01-12
translationKey: "networking-fundamentals"
categories: ["networking"]
math: true
draft: false
---

## Introduction

This article is the result of extensive revision and restructuring. The objective was to present the subject in a clear, direct, and accessible manner. The content is organized into key topics that form a practical and easy-to-assimilate summary. This approach reflects the way the material was intended to be learned, and it is hoped that it will be useful to the reader as well.

The material in this article is aligned with the blueprint for the Cisco CCNA 200-301 certification, providing a solid foundation for those who wish to deepen their study of computer networks.

## Objectives and Network Architecture

### Why do we build networks?

- **Data and service access**: Enable users and applications to access information and services regardless of location.
- **Resource sharing**: Allow shared use of printers, scanners, storage, and other peripherals.
- **Centralized administration**: Facilitate the management, updating, and security of devices and services.

### Pillars of Network Architecture

There are four basic characteristics that network architects must consider in order to meet user expectations:

1. **Fault tolerance**: A network must be resilient, continuing to operate even in the event of partial failures through the use of redundant components and paths.
2. **Scalability**: The ability to grow by adding users and services without performance degradation, while planning for future expansion.
3. **Quality of Service (QoS)**: The ability to prioritize critical traffic, such as voice and video, ensuring the bandwidth, latency, and jitter control required by each application.
4. **Security**: Protect the network and data against unauthorized access (confidentiality), ensure that information has not been altered (integrity), and guarantee that services are always available (availability).

## Fundamental Network Components

### Devices

- **End devices (hosts)**: Where communication originates or terminates, such as computers, servers, smartphones, and IoT devices.
- **Intermediate devices**: Connect end devices and other networks, including switches, routers, and access points. Their functions include regenerating signals, managing routes, applying security policies, and enforcing QoS.

### Transmission Media

- **Metallic (twisted-pair cable)**: Uses electrical pulses.
- **Optical fiber**: Uses light pulses. It is immune to interference and is ideal for high-speed and long-distance links.
- **Wireless**: Uses radio waves, offering mobility.

## Reference Models: A Detailed View

To organize the complexity of communication, reference models were created. They divide network functions into **layers**, where each layer provides a service to the layer above and consumes services from the layer below.

Communication occurs in two ways:

- **Adjacent-layer interaction (vertical)**: On the same device, an upper layer requests a service from the layer below it.
- **Same-layer interaction (horizontal)**: Between different devices, peer layers communicate using protocol headers.

| Concept | Description |
| --- | --- |
| Same-layer interaction | Devices use a protocol to communicate with the same layer on another device. The protocol defines a header that indicates what each device wants to do. |
| Adjacent-layer interaction | On a single device, a lower layer provides a service to the upper layer. The upper-layer software or hardware requests that the lower layer perform the required function. |

### Application Layer

This is the layer closest to the user. It does not define the application itself, but rather the **services and protocols** that applications need in order to interact with the network (HTTP for web, SMTP for e-mail, FTP for file transfer). It acts as the interface between software and the network stack.

### Transport Layer

This layer is responsible for logical end-to-end communication between applications. Its two main protocols are TCP and UDP.

#### Multiplexing and Sockets

So that multiple applications on the same host can communicate simultaneously, the transport layer uses the concept of **ports**. The combination of an IP address and a port number forms a **socket**, which uniquely identifies a communication session.

- **Well-known ports (0-1023)**: Reserved for standard services (80/HTTP, 443/HTTPS, 22/SSH).
- **Ephemeral ports (1024-65535)**: Used dynamically by clients to initiate connections.

#### TCP (Transmission Control Protocol)

TCP is a **reliable, connection-oriented** protocol. It offers the abstraction of a perfect communication channel, even over an unreliable network such as the Internet.

![TCP Header](/assets/images/networking/networking-fundamentals/M1-P2-TCP-HEADER.png)

Its main characteristics are:

- **Ordered and reliable delivery**: Ensures that data arrives in the correct order and without loss, using sequence numbers and acknowledgments (ACKs).
- **Flow control**: Through the **receive window (rwnd)**, the receiver informs the sender of the available buffer space, preventing the sender from transmitting more data than the receiver can process. RFC 1323 introduced window scaling to allow windows larger than 65,535 bytes.
- **Congestion control**: Mechanisms such as **slow start** prevent the network from being overloaded. A **congestion window (cwnd)** limits the amount of data in transit before an ACK is received.
- **TCP Fast Open**: Reduces latency in new connections by reusing information from a previous connection.

#### UDP (User Datagram Protocol)

UDP is a **simple, fast, connectionless** protocol. Its main advantage is the absence of overhead.

![UDP Header](/assets/images/networking/networking-fundamentals/M1-P3-UDP-HEADER.png)

- **No guarantees**: There is no guarantee of delivery, ordering, or flow control. It is a best-effort service.
- **Ideal for**: Real-time applications such as streaming, online gaming, and VoIP, where speed is more critical than reliability.

![TCP vs UDP](/assets/images/networking/networking-fundamentals/M1-P3-TCP-VS-UDP-HEADER.png)

### Network Layer (or Internet Layer)

This layer is responsible for logical addressing (IP) and for routing packets from source to final destination across multiple networks.

IP routing is a collaborative process between hosts and routers. The operating system of the host decides where to send the packet, usually to a nearby router, the default gateway, and subsequent routers make forwarding decisions based on their routing tables.

**Routing Process in a Router:**

1. The router receives a data frame.
2. It checks for errors using the Frame Check Sequence (FCS) field in the trailer. If an error is found, the frame is discarded.
3. It removes the data-link header and trailer, revealing the IP packet.
4. It consults its routing table to find the best route to the destination IP address.
5. It encapsulates the IP packet in a new data-link header and trailer appropriate for the outgoing interface.
6. It forwards the new frame.

### Data Link and Physical Layers (Network Access)

These layers define how data is transmitted over a specific physical medium, such as cable, fiber, or air. They handle physical addressing (MAC address) and error detection on a local link.

#### Ethernet

Ethernet is the most popular LAN technology in the world. It defines:

- **Physical addressing (MAC address)**: A unique 48-bit address burned into the network interface card.
- **Error detection (FCS)**: The Frame Check Sequence field in the frame trailer allows the receiver to verify whether transmission errors occurred. If an error is detected, the frame is discarded. Ethernet detects errors but does not correct them; that responsibility belongs to higher layers, such as TCP.
- **Auto-MDIX**: A feature that automatically detects the cable type, crossover or straight-through, and adjusts the pinout so that the connection works.
- **Operating modes**: **Half-duplex** — cannot transmit and receive at the same time. **Full-duplex** — can transmit and receive simultaneously.

Ethernet has also evolved into a **WAN** technology, with fiber standards that support tens of kilometers, enabling services such as **Ethernet Line Service (E-Line)**.

## Network Types and Topologies

### Scope

- **LAN (Local Area Network)**: A network within a limited geographic area, such as an office or building.
- **WAN (Wide Area Network)**: Connects LANs in geographically distant locations. The Internet is the largest WAN of all.
- **Intranet**: A private network belonging to an organization.
- **Extranet**: Allows controlled access by external partners to parts of the intranet.

### Physical Topologies

These describe how devices are physically connected:

- **Star**: The most common LAN topology, with a central device, usually a switch.
- **Mesh**: Highly redundant topology, common in WANs.
- **Bus/Ring**: Legacy topologies.

## Data Communication Models

### Client/Server vs. Peer-to-Peer

- **Client/Server**: A centralized server provides services to clients. This is the most common model.
- **Peer-to-Peer (P2P)**: All devices are peers and can act as both client and server.

### Circuit Switching vs. Packet Switching

- **Circuit switching**: A dedicated path is established before communication begins, as in traditional telephony. It is inefficient because the channel remains allocated even during silence.
- **Packet switching**: Data is divided into **packets** (or **datagrams**), which are sent independently across the network and may take different paths. This is the model used by the Internet and is more efficient and resilient.

## Conclusion

This article covered the essential fundamentals of computer networks. As observed, a modern network is a complex system that depends on the harmonious interaction of several components:

1. **Physical infrastructure**: Devices, media, and topologies.
2. **Logical architecture**: Layered models, protocols, and services.
3. **Design principles**: Fault tolerance, scalability, QoS, and security.

Networks continue to evolve rapidly, with new technologies constantly emerging. Concepts such as SDN (Software-Defined Networking) and NFV (Network Functions Virtualization) are redefining how networks are designed and managed.

For those who wish to deepen their study of networks, especially in preparation for the CCNA certification, the next topics include:

- **Switching and VLANs**: Understanding local network segmentation and protocols derived from STP.
- **Routing**: Learning the theoretical foundations of dynamic routing protocols.
- **Security**: Understanding firewalls, VPNs, and access control lists.
- **Automation**: Understanding the concepts of automation and programmability tools.

A well-designed network is one that users barely notice; it simply works, allowing people to focus on their activities without concern for the underlying infrastructure.

> "Complexity is your enemy. Any fool can make something complicated. It is hard to make something simple." — Richard Branson

The next articles will explore each of the certification topics in greater depth, building both theoretical and practical knowledge together. Until then, the challenge is to maintain discipline, curiosity, and continued study.
