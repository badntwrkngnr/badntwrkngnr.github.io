---
title: "Introduction to the IPv4 Protocol"
slug: "ipv4-introduction" 
date: 2025-01-15
translationKey: "ipv4-introduction"
categories: ["networking"]
math: true
draft: false
---

## Introduction

This article explores the IP protocol, beginning with part of its historical context and importance, presenting an overview of its functionality, and later delving into the details.

The discussion begins in the Cold War period, when the United States and the Soviet Union were in conflict. The focus here, however, is not the conflict itself, but rather the technological developments that followed and shaped modern computer networks.

The *ARPA* (*Advanced Research Projects Agency*) needed to transmit confidential data between military bases and research departments. From this need emerged the *ARPAnet* (ARPA Network), a communication network that also included universities and some private companies, forming a working group called the *ARPANET Network Working Group*.

Many of the protocols and technologies used today originate from ARPAnet. It started modestly, and over time more institutions connected through dedicated telephone lines.

Initially, the standard protocol for the internet was called NCP (Network Control Program). However, as other networks appeared around the world, compatibility problems arose with existing protocols, leading to the development of a new architecture called the TCP/IP reference model. Its central ideas were:

- Allow routing between different networks.
- Be independent of hardware.
- Enable fault recovery.

To improve understanding of this article, the next section presents the main reference models for computer networks.

## Reference Models for Computer Networks

### OSI Model

The *OSI* (*Open Systems Interconnection*) model was developed by the ISO (*International Organization for Standardization*) out of the need to standardize hardware and protocols. In the beginning, each manufacturer followed a proprietary standard, which made communication between equipment from different vendors difficult or even impossible. The OSI model was proposed to fill these gaps, with the following main objectives:

1. Ensure end-to-end communication.
2. Allow communication between devices from different manufacturers.
3. Define rules for building computer networks regardless of technology or geographic scope.
4. Facilitate the learning of network architecture.
5. Allow new technologies to be easily deployed and updated.

![OSI model](/assets/images/networking/ipv4-introduction/camadas-modelo-osi.png)

The OSI model is structured in seven layers:

1. **Physical**: Electrical and mechanical specifications, representation of bits.
2. **Data Link**: Media access control.
   - Lower sublayer (MAC): media access control.
   - Upper sublayer (LLC): logical link control.
3. **Network**: Packet routing, IP addressing, protocols such as ARP and RARP.
4. **Transport**: Reliable and efficient transport between devices.
   - Connection-oriented protocols (TCP).
   - Unreliable, connectionless protocols (UDP).
5. **Session**: Establishment and management of sessions between applications.
6. **Presentation**: Data format conversion, compression, and encryption.
7. **Application**: User-machine interaction interface, protocols such as HTTP, SMTP, and FTP.

Protocols are associated with layers according to their functionalities.

### TCP/IP Model

The TCP/IP model is composed of a stack of interactive layers, where each layer interacts with the layer above and below in a hierarchical manner. This means that upper-layer protocols depend on lower-layer protocols.

![TCP/IP model](/assets/images/networking/ipv4-introduction/camadas-modelo-tcp-ip.png)

Layers of the TCP/IP model:

1. **Network Access**: Provides support for all proprietary standards.
2. **Internet (or Network)**: Supports the Internet Protocol (IP). Examples: ARP, RARP, and ICMP.
3. **Transport**: Manages the communication session between computers. This layer uses TCP (Transmission Control Protocol) and UDP (User Datagram Protocol).
4. **Application**: TCP/IP application protocols and the interface between user and application. Examples: HTTP, SMTP, FTP, SSH, and others.

### Comparison Between the OSI and TCP/IP Models

![Hybrid model](/assets/images/networking/ipv4-introduction/camadas-modelo-hibrido.png)

**Similarities**: The Transport layers have the same function in both models.

**Differences**: The TCP/IP Link layer combines the functions of the OSI Data Link and Physical layers. The TCP/IP Application layer combines the functions of the OSI Application, Presentation, and Session layers.

**Weakness of OSI**: Implementation complexity and repetition of functionality.

**Weakness of TCP/IP**: Lack of conceptual clarity and limited coverage of other protocol stacks.

For these reasons, Andrew S. Tanenbaum, a researcher and professor in computer science, proposed a five-layer hybrid model. He is also the author of books on operating systems, distributed systems, and computer networks. The goal of this model is to improve upon the deficiencies of the TCP/IP model and eliminate the excesses of the OSI model.

![Model Diff](/assets/images/networking/ipv4-introduction/comparacao-entre-modelos-de-camadas.png)

## Problems Solved by the TCP/IP Protocol Stack

Several topics have been introduced. To facilitate comprehension, especially for readers in the early stages of study, the main problems to be solved were:

1. Waste of resources.
2. Difficulty in scaling services.
3. Lack of resiliency.

To understand this better, it is useful to look back at how communication was originally performed.

First, consider circuit switching. In the beginning, network services used dedicated channels, and to access these services a circuit was created. That is, communication required a predefined path between the source and destination devices. With this technique, network resources remained allocated even when no data was being exchanged, until the connection was terminated.

These problems were addressed through packet switching, but first it is necessary to understand what *packets* are.

> A packet is the *Protocol Data Unit* (PDU) of the network layer, but it can also be understood as a message, or a portion of a larger message, sent over the network.

To understand why packets exist, consider the following: sending complete files across a network is not viable. If an error occurred during data transmission, the entire message could be lost, and transmission would have to start again from the beginning, wasting time and resources. To deal with this problem, data is fragmented into smaller parts and sent across the network. At this stage, it is important to know that there are two ways to send data: with delivery guarantees and without delivery guarantees.

Consider the following analogy for guaranteed delivery. Suppose you have small envelopes that can only hold cards, and you need to send a letter to a friend. The letter must be divided into small parts, marking the sequence so that the friend can reassemble the received messages and read the original letter. It is important to note that, once the messages are separated, there is no guarantee that they will follow the same path to the destination or even arrive in order. This describes the basic operation of packet switching.

Currently, network services do not create dedicated channels for data transmission. It should be noted that this is a simplification, but a detailed discussion of that topic belongs to a separate article. In summary, a packet is, for the most part, a fragment of a message that is transmitted individually, without a predefined route to the destination, and has the advantage of not occupying the communication channel during idle periods. If a route fails, packets can take alternative paths, minimizing downtime. Because there are no dedicated circuits, the network can handle a much larger volume of simultaneous operations.

## Conclusion

After everything that has been presented, the reader may ask: why was the IP protocol developed?

> The IP protocol was designed for use in packet-switched systems, and its scope is to meet the basic needs for delivering data from a source to a destination.

![IPv4 header](/assets/images/networking/ipv4-introduction/cabecalho-ipv4.png)

The IP protocol implements two basic functions: *addressing* and *fragmentation*.

It is important to note that the IP protocol does not provide certain important mechanisms, such as reliability, flow control, sequencing, or error correction. These responsibilities are delegated to the transport and data-link layers.

![Interactions between layers](/assets/images/networking/ipv4-introduction/relacionamento-camadas-tcp-ip.png)

Headers contain the information needed to transmit packets to their destinations. The selection of a path for transmission is called *routing*.

> **Routing**: The process of performing communication between different networks. This process is carried out through interfaces called gateways, which can be viewed as exit points from one network toward others. However, a gateway is more than that; it is responsible for knowing the paths used to forward packets.

An IP datagram is composed of a header plus the *PDU* (protocol data unit) of the transport layer.

![IP datagram](/assets/images/networking/ipv4-introduction/datagrama-ip.png)

The IP protocol uses fields in the internet header to fragment and reassemble *datagrams* when necessary for data transmission.

> Datagrams are basic transfer units that provide unreliable communication service in packet-switched networks. Simplifying, they can be viewed as standalone entities without connections or logical circuits.

The IP protocol operates on every host and gateway to interpret address fields, fragment and assemble datagrams, make routing decisions, and perform other functions.

IP uses four main mechanisms to perform its function:

1. **Type of Service**: A set of parameters that defines the desired quality of service on the network. This helps gateways choose ideal transmission parameters when routing a packet on the Internet.
2. **TTL / Time to Live**: Indicates the maximum lifetime of a packet on the Internet. If this limit is reached (zero), the packet is discarded. It works as a self-destruction mechanism after a certain time, preventing packets from wandering through the network indefinitely.
3. **Options**: Provide additional control functions for specific situations, such as timestamps, security, and special routing. They are generally not used in everyday communication.
4. **Header Checksum**: Protects the fields of the internet header against transmission errors. If the checksum fails, the packet is discarded. Errors are reported through the ICMP (Internet Control Message Protocol).

This article has an introductory character and, even so, it became longer than expected. Some topics will be left for future publications, where the fields of the IP header, addressing, and the relationship between the IP protocol and the transport and data-link layers will be detailed.
