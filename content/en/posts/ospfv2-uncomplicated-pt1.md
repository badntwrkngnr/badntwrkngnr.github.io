---
title: "OSPF Uncomplicated - Part 1"
slug: "ospfv2-uncomplicated-pt1"
date: 2026-05-23
translationKey: "ospfv2-uncomplicated-pt1"
categories: ["networking"]
math: true
draft: false
---

## Introduction

This article is the first part of a series on OSPF. Although the initial intention was to cover the subject in a single article, the breadth of the topic made it advisable to divide it into parts, making the content more accessible. The title follows the naming style used by the Brazilian technology platform [*LinuxTips*](https://linuxtips.io/), of which I am an admirer. This article covers a substantial amount of information; however, for those seeking deeper study, I also recommend the course [*Descomplicando o OSPF*](https://gustavokalau.com.br/) by Professor Gustavo Kalau (note: the course is in Portuguese).

Before addressing OSPF, let us review the role of a dynamic routing protocol. These concepts are simple, but they are fundamental, and they prepare the ground for what follows.

The main functions of a routing protocol include:

- **Neighbor and network discovery:** Automatic identification of other routers on the same segment, eliminating the need to map routes manually every time the network expands.
- **Best path calculation and selection:** Use of algorithms that analyze the topology (bandwidth, delay, number of routers, etc.) to choose the best route for each prefix.
- **Loop prevention:** Mechanisms that prevent packets from circulating indefinitely. In link-state protocols such as OSPF, all routers calculate routes from a loop-free topological map.
- **Fault tolerance:** Reaction to link or device failures, recalculating alternative routes.
- **Classless routing:** Support for VLSM and CIDR by carrying subnet masks in routing updates.

## Core Concepts

Let us now examine OSPFv2. OSPFv2 (Open Shortest Path First version 2) is an internal dynamic routing protocol (IGP). It is a link-state protocol, standardized by the IETF in RFC 2328. To understand what this means, let us compare it with older protocols such as RIP (a distance-vector protocol):

- **Distance vector:** This type of routing uses the number of hops (routers traversed) as the criterion for choosing the best path: the fewer hops, the better. It is analogous to driving while only reading distance signs: one trusts the numbers without knowing whether the road is congested or in poor condition. In real networks, interfaces operate at different speeds, so the path with the fewest hops may be the slowest and, paradoxically, the "best" path may actually be the worst.
- **Link-state:** Routers exchange information as if they were pieces of a puzzle. In OSPF, these messages are called LSAs (Link-State Advertisements). When assembled in the LSDB (Link-State Database) and synchronized, the puzzle is complete, and **each router builds an identical and complete topological map of the area**: all nodes, links, status (Up/Down), and the "cost" of each segment.

The OSPF lifecycle in an area can be summarized in three phases:

1. Neighbor discovery and adjacency formation.
2. Construction and synchronization of the LSDB.
3. Execution of the SPF (Dijkstra) algorithm.

### SPF and Metric

With the network map (LSDB) ready, OSPF finds the best path using the SPF (Shortest Path First) algorithm, created by Edsger W. Dijkstra. The calculation has four steps:

**1. Starting point:** Each router runs SPF independently, placing itself at the top (the root) of the Shortest Path Tree (SPT).

**2. Metric:** The criterion for choosing the best path is the **cost**, which is inversely proportional to the link's bandwidth: faster links have lower costs.

![OSPF Cost Formula](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-11.png)

OSPF considers the conditions of the path, but it is extremely important to note that it uses only **bandwidth** as the parameter for calculating its metric. Another important detail: by default, Cisco IOS uses a reference bandwidth of 100 Mbps, which causes both FastEthernet (100 Mbps) and GigabitEthernet (1000 Mbps) interfaces to receive the same cost of **1**. This can be problematic; to correct it, use the **auto-cost reference-bandwidth** command on all devices in the OSPF domain. This will be demonstrated in the troubleshooting material. As a side note, the EIGRP protocol can consider bandwidth, delay, load, and reliability.

**3. Cumulative cost:** The router sums the outgoing interface cost along the path to each destination.

**4. RIB installation:** For the same subnet, the path with the lowest cumulative cost wins. The losing path remains only in the LSDB. The winning route is installed in the **RIB**. In case of a tie, OSPF installs both paths and balances traffic (ECMP, Equal-Cost Multipathing).

To demonstrate the behavior of the two algorithm types mentioned above, consider the topology below:

![RIP and OSPF Topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1.png)

Let us first illustrate the operation of a protocol that uses the distance-vector algorithm, RIP:

- As a starting point, let us say that the blue connections are Ethernet links and the yellow connections are Serial links.
- Imagine that a packet must travel from router RT2 to RT7.
- Based on the topology shown, the shortest path would be RT2 > RT4 > RT7.
- Given the speed of the links, this "shortest path" may not be the most efficient (fastest) way to reach the destination.

![RIP path in the topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-1.png)

Let us now examine the basic operation of OSPF, which uses link-state logic. Using the cost formula, the costs can be defined as shown in the image below:

![OSPF costs in the topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-2.png)

Now that OSPF has calculated the accumulated cost from the source to the destination, we can observe the packet-forwarding behavior in practice.
OSPF is already enabled in this topology; the procedures for enabling it will be explained in the adjacency section. The lab files are available for [*download*](https://drive.google.com/drive/folders/1PFSQtfXWONkJiUJ5dYLB7eKL-c4M5bcl?usp=drive_link); you may download them all and practice as much as needed. I use PNETLab, and the files are also fully compatible with EVE-NG.

When examining the routing table, we can see that the route to the destination network located on router RT7 has a cost of 8. One might ask why the cost is 8, given that the sum of the outgoing interface costs equals 7. This is a subtle detail that can cause confusion. As defined in the OSPF RFC and implemented by Cisco IOS, the loopback interface is treated as a *stub host* and always receives a fixed automatic cost of 1. In the context of networking, a stub can be compared to a dead-end street: traffic may enter and leave, but it never continues on directly to another destination. This should not be confused with a *stub area*, which is a different concept that will be addressed in future articles.

```cisco
RT2#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/2] via 10.1.12.1, 00:00:16, Ethernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback0
      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/3] via 10.1.12.1, 00:00:16, Ethernet0/0
      4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/4] via 10.1.12.1, 00:00:06, Ethernet0/0
      5.0.0.0/32 is subnetted, 1 subnets
O        5.5.5.5 [110/7] via 10.1.12.1, 00:00:06, Ethernet0/0
      6.0.0.0/32 is subnetted, 1 subnets
O        6.6.6.6 [110/5] via 10.1.12.1, 00:00:06, Ethernet0/0
      7.0.0.0/32 is subnetted, 1 subnets
O        7.7.7.7 [110/8] via 10.1.12.1, 00:00:06, Ethernet0/0
      8.0.0.0/32 is subnetted, 1 subnets
O        8.8.8.8 [110/6] via 10.1.12.1, 00:00:06, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 11 subnets, 2 masks
C        10.1.12.0/24 is directly connected, Ethernet0/0
L        10.1.12.2/32 is directly connected, Ethernet0/0
O        10.1.13.0/24 [110/2] via 10.1.12.1, 00:00:16, Ethernet0/0
C        10.1.24.0/24 is directly connected, Serial1/0
L        10.1.24.2/32 is directly connected, Serial1/0
O        10.1.34.0/24 [110/3] via 10.1.12.1, 00:00:16, Ethernet0/0
O        10.1.46.0/24 [110/4] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.47.0/24 [110/67] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.58.0/24 [110/6] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.68.0/24 [110/5] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.75.0/24 [110/7] via 10.1.12.1, 00:00:06, Ethernet0/0
```

Here is a picture showing the path the packets take to reach the destination:

![Path taken by packets to the destination](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-4.png)

The *traceroute* command below shows the actual path the packets travel:

```cisco
RT2#traceroute 7.7.7.7
Type escape sequence to abort.
Tracing the route to 7.7.7.7
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.12.1 1 msec 1 msec 1 msec
  2 10.1.13.3 1 msec 1 msec 0 msec
  3 10.1.34.4 1 msec 2 msec 1 msec
  4 10.1.46.6 1 msec 2 msec 1 msec
  5 10.1.68.8 2 msec 2 msec 2 msec
  6 10.1.58.5 1 msec 2 msec 1 msec
  7 10.1.75.7 2 msec 2 msec *
RT2#
```

### Neighbor Discovery and Adjacency Formation

Discovering a neighbor (Phase 1) is only the first step. However, before routers begin exchanging information and going through the state machine, they must agree on a set of parameters. Through Hello packets, they validate a checklist of requirements. If any item does not match, the adjacency formation process will not even begin.

For the adjacency formation process to start, the following settings must be identical on both sides of the link:

- **Area ID:** If one router belongs to Area 0 and the other belongs to Area 1 on the same link, they will not proceed with adjacency formation.
- **Subnet and Mask:** The IP addresses of the connected interfaces must belong to the exact same network.
- **Hello and Dead Timers:** The timers must match (usually 10 seconds for Hello and 40 seconds for Dead).
- **Authentication:** If a password is configured, it must be identical on both sides.
- **Special Area Flags (Stub/NSSA):** Both routers must agree on the type of area in which they are operating.

In addition to these, there are two more rules to ensure that the process does not stall later, during database exchange:

- **Unique Router IDs:** Each router needs its own identity (RID). If there is a duplicate in the network, the routing domain can become unstable.
- **Matching MTU (Maximum Transmission Unit):** The maximum packet size supported by the interfaces must be the same. If one side has an MTU of 1500 bytes and the other 1400, they will begin to communicate, but the adjacency will stall halfway. This occurs because OSPF does not support fragmentation.

> The OSPF process number does not need to be identical between routers to form an adjacency.

To show this better, we will use this topology:

![OSPF adjacency topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-6.png)

With all of this previously aligned, to synchronize the LSDB the OSPF routers go through a seven-step state machine. Communication occurs directly over IP (protocol ID 89), with five packet types — **Hello**, **DBD**, **LSR**, **LSU**, and **LSAck** — appearing as the states advance:

1. **Down (inactive):** The initial state. The OSPF process is active on the interface, but no Hello has been received. The router sends Hellos to the multicast address **224.0.0.5** (*All OSPF Routers*) and waits for a response.
2. **Init (initialization):** A Hello arrives from a neighbor, but communication is still one-way. The local RID does not appear in the active-neighbor field of the received packet.
3. **2-Way (bidirectional):** The Hello lists the router's own **Router ID (RID)**. Bidirectional communication is confirmed. The protocol evaluates the interface's **Network Type** and decides the next step. In this article, only the two most common types are addressed: **Point-to-Point**, connections between two nodes, in which the DR/BDR election is skipped and the process continues to the following states; and **Broadcast (multi-access)**, the default on Ethernet. With several devices on the segment, OSPF elects a *Designated Router* (DR) and a *Backup Designated Router* (BDR) to centralize LSA exchange. The remaining routers, called DROTHERs, maintain a **2-Way** state with one another and only reach the **Full** state with the DR and BDR. This avoids scalability problems.
4. **ExStart:** Basic communication is ready; only LSDB synchronization remains. The peers elect a *Master* and a *Slave* (the higher RID wins) to define the initial sequence of **DBD** (*Database Description*) packets.
5. **Exchange:** Master and Slave exchange DBDs, the "index" of the LSDB, with LSA headers for comparison, but still without the full content of the routes.
6. **Loading:** After the DBDs, the router requests missing or newer LSAs via **LSR** (*Link-State Request*). The neighbor responds with **LSU** (*Link-State Update*), and the requester confirms with **LSAck** (*Link-State Acknowledgment*).
7. **Full:** The LSDBs are identical and synchronized for the area. The router now runs SPF and installs the best routes into the routing table (RIB).

> Note: Phases 2 and 3 happen in the **Exchange**, **Loading**, and **Full** states. The LSDB is shared. Then, each router runs SPF independently to build the RIB.

### Enabling OSPF on Cisco IOS

Although a considerable amount of theory has been presented, practical exercise is essential for a complete understanding. Before any Hello packet is sent, OSPF must be enabled on the routers. In Cisco IOS, there are two ways to do this:

#### 1. Network statement (global configuration under *router ospf*)

This is the traditional model. You enter the OSPF process configuration and specify which networks (and, consequently, which interfaces) will participate in the protocol. The configuration below was applied on router *RT1*:

```cisco
router ospf 1
 network 10.0.0.1 0.0.0.0 area 0
```

It is common to confuse the *network* statement with a route advertisement command; however, its function is somewhat different. In practice, it acts as an interface selector. The process is as follows:

- You provide an IP address and a wildcard mask.
- The router checks its configured interfaces.
- If the interface IP address matches the wildcard mask, OSPF begins to operate on it.

That is all.

In the example of IP address 10.0.0.1 with wildcard mask 0.0.0.0, we are being very specific: only the interface with that exact IP will participate in OSPF. If the address is not found, the interface does not enter the process and, consequently, no route is advertised. Understanding this concept is fundamental and can save time when analyzing network behavior.

#### 2. Interface-level configuration

The alternative is to enable OSPF directly on the interface, without applying a *network* statement, as was done on router *RT2*:

```cisco
interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip ospf 2 area 0
```

Both methods enable OSPF on the interface. The only difference is the configuration location: in the process (generic/centralized) or directly on the interface (specific).

> On Cisco IOS, specific configurations take precedence over generic configurations. This is no different for OSPF. In the event of a conflict — for example, if a *network* statement places the interface in Area 0 while an *ip ospf* command on the interface places it in Area 1 — the interface-level configuration takes precedence. This will be demonstrated in the troubleshooting article.

##### Configuration Tip — Network Type

It is worth remembering the default OSPF behavior based on network type. In our topology, the command *ip ospf network point-to-point* should be applied to the interface, because it is a point-to-point connection, in order to override the default *broadcast* behavior of Ethernet. With this, there is no unnecessary DR/BDR election. We will observe this in practice shortly.

#### Returning to the main subject

Let us now observe the routers becoming neighbors:

- Once OSPF is configured, the router sends Hello packets, as can be seen in the Wireshark captures below:

![OSPF Hello packet capture](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-8.png)

![OSPF Hello packet detail](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-9.png)

![OSPF Hello packet in capture](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-10.png)

The **show ip protocols** command was used on RT1 to view the active routing protocols:

```cisco
RT1#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 10.0.0.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.1 0.0.0.0 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 110)
```

After the connection is complete, the **show ip ospf neighbor** command shows this:

```cisco
RT1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.2          1   FULL/DR         00:00:38    10.0.0.2        Ethernet0/0
```

Now observe the *show ip protocols* output on router RT2:

```cisco
RT2#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 2"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 10.0.0.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 0):
    Ethernet0/0
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 110)
```

And RT2 after the adjacency is complete:

```cisco
RT2#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.1          1   FULL/BDR        00:00:36    10.0.0.1        Ethernet0/0
```

The difference in how OSPF was configured can be seen by examining the **show ip protocols** output:

- **RT1** via *network statement*

```cisco
Routing for Networks:
  10.0.0.1 0.0.0.0 area 0
```

- **RT2** via *interface*

```cisco
Routing on Interfaces Configured Explicitly (Area 0):
  Ethernet0/0
```

With *debug ip ospf adj* enabled, the state transitions can be observed step by step on **RT1**:

```cisco
*Jun  7 01:41:48.306: OSPF-1 ADJ   Et0/0: Interface going Up
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: end of Wait on interface
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.1
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect BDR 0.0.0.0
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: none
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: 2 Way Communication to 10.0.0.2, state 2WAY
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Neighbor change event
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Prepare dbase exchange
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0x2376 opt 0x52 flag 0x7 len 32
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Neighbor change event
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Neighbor change event
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0xD30 opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: NBR Negotiation Done. We are the SLAVE
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Summary list built, size 1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0xD30 opt 0x52 flag 0x2 len 52
*Jun  7 01:44:54.416: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0xD31 opt 0x52 flag 0x1 len 32  mtu 1500 state EXCHANGE
*Jun  7 01:44:54.416: OSPF-1 ADJ   Et0/0: Exchange Done with 10.0.0.2
*Jun  7 01:44:54.416: OSPF-1 ADJ   Et0/0: Synchronized with 10.0.0.2, state FULL
*Jun  7 01:44:54.416: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.0.2 on Ethernet0/0 from LOADING to FULL, Loading Done
```

And on **RT2**:

```cisco
*Jun  7 01:44:54.413: OSPF-2 ADJ   Et0/0: Interface going Up
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: 2 Way Communication to 10.0.0.1, state 2WAY
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Backup seen event before WAIT timer
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Prepare dbase exchange
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0xD30 opt 0x52 flag 0x7 len 32
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0x2376 opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: First DBD and we are not SLAVE
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0xD30 opt 0x52 flag 0x2 len 52  mtu 1500 state EXSTART
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: NBR Negotiation Done. We are the MASTER
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Summary list built, size 0
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0xD31 opt 0x52 flag 0x1 len 32
*Jun  7 01:44:54.416: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0xD31 opt 0x52 flag 0x0 len 32  mtu 1500 state EXCHANGE
*Jun  7 01:44:54.416: OSPF-2 ADJ   Et0/0: Exchange Done with 10.0.0.1
*Jun  7 01:44:54.416: OSPF-2 ADJ   Et0/0: Send LS REQ to 10.0.0.1 length 36 LSA count 1
*Jun  7 01:44:54.417: OSPF-2 ADJ   Et0/0: Rcv LS UPD from 10.0.0.1 length 64 LSA count 1
*Jun  7 01:44:54.417: OSPF-2 ADJ   Et0/0: Synchronized with 10.0.0.1, state FULL
RT2(config-router)#
*Jun  7 01:44:54.417: %OSPF-5-ADJCHG: Process 2, Nbr 10.0.0.1 on Ethernet0/0 from LOADING to FULL, Loading Done
```

Thus far, the **broadcast** network behavior, including DR/BDR election, has been presented. To avoid an unnecessary DR/BDR election, the network type can be changed to *point-to-point* directly on the interface. The effect on the process is shown below:

- **RT1**

```cisco
interface Ethernet0/0
 ip address 10.0.0.1 255.255.255.252
 ip ospf network point-to-point
```

- **RT2**

```cisco
interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip ospf network point-to-point
```

**RT1** logs after the change:

```cisco
*Jun  7 02:35:37.050: OSPF-1 ADJ   Et0/0: Interface going Up
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: 2 Way Communication to 10.0.0.2, state 2WAY
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Prepare dbase exchange
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0xDFC opt 0x52 flag 0x7 len 32
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0x92C opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: NBR Negotiation Done. We are the SLAVE
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Summary list built, size 1
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0x92C opt 0x52 flag 0x2 len 52
*Jun  7 02:36:01.280: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0x92D opt 0x52 flag 0x1 len 32  mtu 1500 state EXCHANGE
*Jun  7 02:36:01.280: OSPF-1 ADJ   Et0/0: Exchange Done with 10.0.0.2
*Jun  7 02:36:01.280: OSPF-1 ADJ   Et0/0: Synchronized with 10.0.0.2, state FULL
*Jun  7 02:36:01.280: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.0.2 on Ethernet0/0 from LOADING to FULL, Loading Done
```

**RT2** logs:

```cisco
*Jun  7 02:36:01.278: OSPF-2 ADJ   Et0/0: Interface going Up
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: 2 Way Communication to 10.0.0.1, state 2WAY
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Prepare dbase exchange
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0x92C opt 0x52 flag 0x7 len 32
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0xDFC opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: First DBD and we are not SLAVE
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0x92C opt 0x52 flag 0x2 len 52  mtu 1500 state EXSTART
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: NBR Negotiation Done. We are the MASTER
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Summary list built, size 0
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0x92D opt 0x52 flag 0x1 len 32
*Jun  7 02:36:01.280: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0x92D opt 0x52 flag 0x0 len 32  mtu 1500 state EXCHANGE
*Jun  7 02:36:01.280: OSPF-2 ADJ   Et0/0: Exchange Done with 10.0.0.1
*Jun  7 02:36:01.280: OSPF-2 ADJ   Et0/0: Send LS REQ to 10.0.0.1 length 36 LSA count 1
*Jun  7 02:36:01.281: OSPF-2 ADJ   Et0/0: Rcv LS UPD from 10.0.0.1 length 64 LSA count 1
*Jun  7 02:36:01.281: OSPF-2 ADJ   Et0/0: Synchronized with 10.0.0.1, state FULL
```

Notice there's no DR/BDR anymore:

- **RT1**

```cisco
RT1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.2          0   FULL/  -        00:00:35    10.0.0.2        Ethernet0/0
```

- **RT2**

```cisco
RT2#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.1          0   FULL/  -        00:00:38    10.0.0.1        Ethernet0/0
```

## Features and Capabilities

- **Areas:** The network is divided into areas to isolate failures and conserve CPU resources. Area 0 is the backbone; all other areas must connect to it, either physically or through a virtual link, to exchange routes.
- **Router ID:** A unique 32-bit identifier for the router. It has the same format as an IPv4 address, but it is not a usable address for user traffic.
- **DR and BDR:** On shared networks (such as Ethernet switches), OSPF elects a Designated Router (DR) and a Backup Designated Router (BDR) to centralize LSA exchange. DROTHERs report changes to the DR, which replicates the information on the segment.
- **Summarization:** At area boundaries (ABRs) or during redistribution (ASBRs), prefixes are aggregated to stabilize the LSDB. These roles will be detailed in the next article in the series.
- **Area types (Stub, Totally Stubby, NSSA):** Special areas that do not require complete external routes; instead, they receive a default route, saving CPU and memory. Remember the loopback stub host? Here the concept is different: we are talking about an OSPF area type, not an isolated host.
- **Authentication:** Clear-text or encryption (MD5, HMAC-SHA) to prevent unauthorized routers from joining the domain.

### Configuration Tip — The Router ID

As observed, the Neighbor ID initially corresponds to the IP address of the interface on which OSPF was enabled. In fact, OSPF is not concerned with the device's hostname. The protocol identifies each device through a Router ID (RID), a 32-bit identifier (with the same format as an IP address) that must be absolutely unique within the OSPF domain.

To select this identifier, the OSPF process follows a very specific hierarchy:

1. **Manual configuration:** If the administrator configures the router-id manually, the protocol obeys immediately and ignores the remaining options.
2. **Highest IP address on a Loopback interface:** In the absence of a manual configuration, OSPF checks all active Loopback interfaces and chooses the one with the highest IP address.
3. **Highest IP address on a Physical interface:** If the previous options are not available, it looks for the active physical interface with the highest IP address. This is not recommended, because if that interface fails, the OSPF process can become unstable.

Configuring the RID manually prevents future issues and ensures stability. Below are the commands applied on routers RT1 and RT2 to set the Router ID manually:

```cisco
RT1(config)# router ospf 1
RT1(config-router)# router-id 1.1.1.1
```

```cisco
RT2(config)# router ospf 2
RT2(config-router)# router-id 2.2.2.2
```

## Conclusion: The End of the Beginning

We have reached the end of this first chapter. As we have seen, OSPF does not advertise routes indiscriminately; it is methodical. It validates neighbors carefully through Hello packets and constructs a complete topological map using Dijkstra's algorithm before making any routing decisions.

Although it may seem repetitive, understanding these fundamentals — the rules for forming adjacencies, the state machine, and cost calculations — is exactly what distinguishes a true network engineer from someone who merely copies commands. When something goes wrong, it is this foundational knowledge that enables effective troubleshooting.

In Part 1, we worked with laboratories, packet captures, traceroute, configurations, and the log messages that indicate the FULL state. However, there is still much more to explore. Theory and practice must go hand in hand.

Part 2 will explore the subject in greater depth, covering Hello packets, LSA types, more complex topologies, stub and NSSA areas, summarization, and practical troubleshooting. We will also demonstrate the *auto-cost reference-bandwidth* command and the precedence behavior when global and interface-level configurations conflict. It is hoped that the foundation presented here will be useful.

Therefore, choose your preferred emulator (Packet Tracer, GNS3, EVE-NG, or PNETLab) and continue to the next article in the series. Remember that the lab files are available for download, and that the interfaces are pre-configured.

Together, let us master OSPF.
