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

Hey, how's it going? This was supposed to be one single article covering a topic from start to finish. But as I kept writing, it just kept growing! I have a very hard time knowing when to stop. Halfway through, I decided to change my approach and split it into parts so it wouldn't become a complete slog to get through. I chose this name because I'm a big fan of a Brazilian tech platform called [*LinuxTips*](https://linuxtips.io/), which uses this naming style for their courses. This article covers a lot of information, but I also highly recommend the course [*Descomplicando o OSPF*](https://gustavokalau.com.br/) by Professor Gustavo Kalau (note: the course is in Portuguese!) if you want to learn even more.

Before we get into OSPF itself, it's worth recapping what a dynamic routing protocol actually does. These are simple concepts, but they're fundamental and they'll set the stage for everything that follows.

The main functions of a routing protocol include:

- **Neighbor and network discovery:** automatic (or manual) identification of other routers on the same segment, eliminating the need to manually map routes every time the network grows.
- **Best path calculation and selection (*path selection*):** algorithms that analyze the topology (bandwidth, delay, hop count, etc.) to choose the best route per prefix.
- **Loop prevention:** mechanisms that stop packets from circulating indefinitely. In *link-state* protocols like OSPF, all nodes calculate routes from a loop-free topological map.
- **Fault tolerance:** reacting to link or device failures by recalculating alternative routes.
- **Classless routing (*classless*):** support for VLSM and CIDR, carrying subnet masks in routing updates.

## Core Concepts

Let's start talking about OSPFv2. OSPFv2 (*Open Shortest Path First version 2*) is an internal dynamic routing protocol (IGP) of the *link-state* type, standardized by the IETF in RFC 2328. To understand what that actually means, let's compare it to older protocols like RIP (*distance vector*):

- **Distance vector:** routing uses hop count as its criterion (fewer hops = better). It's like driving by only looking at distance signs: you trust the number, but you have no idea what the road is like, whether there's traffic, or how the streets connect. In practice, interfaces run at different speeds, so the path with fewer hops can be the slowest one — and paradoxically, the "best" path can turn out to be the worst.
- **Link-state:** routers exchange information like puzzle pieces (in OSPF, these messages are LSAs, *Link-State Advertisements*). Assembled in the LSDB (*Link-State Database*), once synchronized, the puzzle is complete and **each router builds an identical and complete topological map of the area**: all nodes, links, status (Up/Down), and the "toll" (cost) of each segment.

The OSPF lifecycle within an area comes down to three phases:

1. Neighbor discovery and adjacency formation
2. Building and synchronizing the LSDB
3. Running the SPF algorithm (*Dijkstra*)

### SPF and Metric

With the network map (LSDB) in hand, OSPF finds the best path using the **SPF (*Shortest Path First*)** algorithm, by Edsger W. Dijkstra. The calculation follows four steps:

**1. Starting point:** each router runs SPF independently, placing **itself** as the root of the *Shortest Path Tree* (SPT).

**2. Metric:** the best-path criterion is **cost**, which is inversely proportional to the link's bandwidth — faster links, cheaper toll.

![OSPF Cost Formula](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-11.png)

OSPF takes path conditions into account, but pay close attention to one extremely important detail: it only considers **bandwidth** as the parameter for metric calculation. Another key point: by default, the reference bandwidth is 100 Mbps on Cisco IOS, which makes FastEthernet and GigabitEthernet ports share the same cost of **1** — that can be a problem. To fix it, use the **auto-cost reference-bandwidth** command on all devices in the OSPF domain to avoid distorting the calculation. We'll demonstrate this in the troubleshooting article. Just as a side note, EIGRP can factor in bandwidth, delay, load, and reliability.

**3. Cumulative cost:** the router sums the outgoing interface cost along the path to each destination.

**4. RIB installation:** for the same subnet, the lowest cumulative cost wins — the loser stays in the LSDB only. The winning route goes into the **RIB**. In case of a tie, OSPF installs both paths and load-balances (ECMP, *Equal-Cost Multipathing*).

To demonstrate the behavior difference between the two algorithm types mentioned above, let's use the topology below:

![RIP and OSPF Topology](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1.png)

Let's start by walking through how a *Distance Vector* protocol like RIP works:

- As a starting point, let's say the blue connections are *Ethernet Links* and the yellow ones are *Serial Links*.
- Now imagine a packet needs to travel from router RT2 to RT7.
- Based on the topology we have, the shortest path would be RT2 > RT4 > RT7.
- Based on link speeds, the "shortest path" might not actually be the most efficient (fastest) way to reach the destination.

![RIP path in the topology](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-1.png)

Now let's walk through the basic behavior of OSPF, which uses *link-state* logic. Using the cost formula provided, we can define the costs as shown in the topology below:

![OSPF costs in the topology](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-2.png)

Now, with the cumulative cost from source to destination calculated by OSPF, let's see how packet forwarding actually behaves in practice!
The topology already has OSPF enabled — we'll explain both activation methods in the adjacency section. The lab files are available for [*download*](https://drive.google.com/drive/folders/1PFSQtfXWONkJiUJ5dYLB7eKL-c4M5bcl?usp=drive_link). You can download all of them and practice as much as you want. I use PNETLab, and the files are fully compatible with EVE-NG!

When we examine the routing table, we can see that we're receiving the destination route for the network on router RT7 with a cost of 8. You might be wondering how, since if we add up the outgoing interface costs to the destination, we get 7. Here's a tiny detail that can throw off your analysis of a routing table. As defined in the OSPF RFC and implemented by Cisco IOS, the loopback interface is treated as a *stub host* and always gets a fixed, automatic cost of 1. I'm forced to bring up the term *stub* a bit early, but to understand the concept, think of it in a networking context as a "dead end" — a device where traffic can come in and go out normally, but will never pass directly through to a third destination because there's only one path in and out. Don't confuse it with a *stub area*, which is a different concept — we'll cover that in upcoming articles.

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

Here's a visual representation of the path packets take to reach the destination:

![Path taken by packets to the destination](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-4.png)

Below we can observe, through the *traceroute* command, the path packets travel to reach the destination network:

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

Discovering a neighbor (phase 1) is just the beginning. But before routers start exchanging information and going through the state machine, they need to agree on a sort of "gentleman's agreement." Through Hello packets, they validate a checklist. If anything is off, the neighbor process doesn't even get started!

For the adjacency formation process to begin, the following settings must be exactly identical on both sides of the link:

- **Area ID:** If one router claims to belong to Area 0 and the other to Area 1 on the same link, they won't move forward with the adjacency process.
- **Subnet and Mask:** The IP addresses of the connected interfaces must belong to exactly the same network.
- **Hello and Dead Timers:** The conversation rhythm must match (typically 10s for Hello and 40s for the Dead Timer).
- **Authentication:** If a password is configured, it must be identical on both sides.
- **Special Area Flags (Stub/NSSA):** Both routers must agree on the type of area they're in.

In addition to these, there are two rules to ensure the process doesn't stall later during database exchange:

- **Unique Router IDs:** Each router needs its own unique identity (RID). If there's a duplicate in the network, the routing table collapses.
- **Matching MTU (Maximum Transmission Unit):** The maximum packet size supported on the interfaces must be the same. If one side has an MTU of 1500 bytes and the other 1400, they'll start talking, but the adjacency will stall midway through. This happens because OSPF doesn't support fragmentation.

> The OSPF process number does NOT need to be identical between routers to form an adjacency.

To better illustrate this, here's the topology we'll use:

![OSPF adjacency topology](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-6.png)

With all that aligned, to synchronize the LSDB, OSPF routers go through a seven-step state machine. Communication happens directly over IP (protocol ID 89), with five packet types — **Hello**, **DBD**, **LSR**, **LSU**, and **LSAck** — which appear as the states progress:

1. **Down:** initial state. The OSPF process is active on the interface but no *Hello* has been received. The router sends *Hello* to multicast **224.0.0.5** (*All OSPF Routers*) and waits for a response.
2. **Init:** a *Hello* arrived from a neighbor, but communication is still one-directional. The local RID doesn't appear in the active neighbor list of the received packet.
3. **2-Way:** the *Hello* lists the router's own *Router ID (RID)*. Bidirectional communication is confirmed. The protocol evaluates the interface's **Network Type** and decides the next step. We'll focus on the two most common types here: **Point-to-Point**, connections between two nodes — DR/BDR election is skipped and the process moves to the next states. **Broadcast (multi-access)**, the default on Ethernet — with multiple devices on the segment, OSPF elects a *Designated Router* (DR) and a *Backup Designated Router* (BDR) to centralize LSA exchange. The rest (*DROTHERs*) maintain **2-Way** adjacency with each other and only advance to **Full** with the DR and BDR. We'll detail this in the Features section — this happens to avoid scalability issues.
4. **ExStart:** basic communication is ready, now the LSDB needs to be synchronized. The peers elect *Master* and *Slave* (highest RID wins) to define the initial sequence of **DBD** (*Database Description*) packets.
5. **Exchange:** *Master* and *Slave* swap DBDs — the "index" of the LSDB — with LSA headers for comparison, but without the full route content yet.
6. **Loading:** after the DBDs, the router requests missing or newer LSAs via **LSR** (*Link-State Request*). The neighbor responds with **LSU** (*Link-State Update*) and the requester confirms with **LSAck** (*Link-State Acknowledgment*).
7. **Full:** the LSDBs are identical and synchronized across the area — ***LET'S GOOOOO!*** The router finally runs SPF and installs the best routes into the routing table (RIB).

> Note: Phases 2 and 3 happen in the **Exchange**, **Loading**, and **Full** states: the LSDB gets synchronized, then each router runs SPF independently to populate the RIB.

### Enabling OSPF on Cisco IOS

We've covered a lot of theory, but hands-on practice is essential for real understanding. Before any *Hello* packet goes out on the network, we need to enable OSPF on the routers. In Cisco IOS, there are two ways to do this:

#### 1. Network statement (global configuration under *router ospf*)

This is the classic approach. You enter the OSPF process configuration mode and specify which networks (and therefore which interfaces) will participate in the protocol. The configuration below was applied on router *RT1*:

```cisco
router ospf 1
 network 10.0.0.1 0.0.0.0 area 0
```

A lot of people confuse the network statement with a route advertisement command, but it works a bit differently — it acts as an **interface selector**. The process is straightforward:

- You provide an address and a wildcard mask.
- The router iterates through its configured interfaces.
- If the interface's IP address matches the wildcard, OSPF becomes active on it.

That's it!

In the example of IP address 10.0.0.1 with wildcard mask 0.0.0.0, we're being very specific — only the interface with that exact IP will participate in OSPF. If the address isn't found, the interface doesn't join the process and, consequently, no route is advertised. Understanding this concept is fundamental and can save you a lot of time when troubleshooting network behavior.

#### 2. Interface-level configuration

The alternative is to attach the interface directly to the OSPF process, without needing a *network statement*, as was done on router *RT2*:

```cisco
interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip ospf 2 area 0
```

Both approaches activate OSPF on the interface. The only difference is **where** you configure it: in the process (generic/centralized) or on the interface (specific).

> On Cisco IOS, **specific** configurations take precedence over **generic** ones — and OSPF is no different. If there's a conflict — for example, a *network statement* placing the interface in Area 0 and an *ip ospf 2 area 1* on the same port — the interface-level config wins. We'll demonstrate this in the upcoming troubleshooting article. I need to stay focused and not let this material grow any more than it already has!

##### First round of refinement on the device configuration

It's also worth noting the default OSPF behavior based on network type. In our topology, we should apply the *ip ospf network point-to-point* command on the interface, since it's a point-to-point connection, to override the default *broadcast* behavior that Ethernet has — this prevents an unnecessary DR/BDR election. We'll see this in action shortly.

#### Back to the main topic

Now let's watch the neighbor and adjacency formation process in action:

- Once we configure OSPF, the router starts sending Hello packets, as seen in the Wireshark captures below:

![OSPF Hello packet capture](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-8.png)

![OSPF Hello packet detail](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-9.png)

![OSPF Hello packet in capture](/assets/images/networking/ospfv2-descomplicado/ospfv2-descomplicado-pt1-10.png)

The command below was applied to check which routing protocols are active on router *RT1*:

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

After the adjacency process completes, the *show ip ospf neighbor* command should show the output below:

```cisco
RT1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.2          1   FULL/DR         00:00:38    10.0.0.2        Ethernet0/0
```

Now let's check the *show ip protocols* output on router *RT2*:

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

And *RT2* after the adjacency is complete:

```cisco
RT2#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.1          1   FULL/BDR        00:00:36    10.0.0.1        Ethernet0/0
```

We can spot the difference between how OSPF was enabled on each router by paying attention to the *show ip protocols* output:

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

With *debug ip ospf adj* enabled, you can follow the state transitions on **RT1**:

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

So far we've seen the **broadcast** network behavior, with DR/BDR election. To avoid an unnecessary DR/BDR election, let's change the network type to *point-to-point* directly on the interface and see how the process changes:

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

- **Areas:** the topology is divided into areas to isolate failures and reduce SPF runs. Area 0 is the *backbone* — all other areas must connect to it (either physically or via *virtual link*) to communicate with each other.
- **Router ID:** a unique 32-bit identifier per router (same format as an IPv4 address, but not a routable address for user traffic).
- **DR and BDR:** on broadcast segments, OSPF elects a *Designated Router* and a *Backup Designated Router* to avoid full adjacencies between all routers. *DROTHERs* report changes to the DR, which propagates the information across the segment.
- **Summarization:** at area boundaries (ABRs) or during redistribution (ASBRs), condensing prefixes to stabilize the LSDB. We'll detail those roles in the next article.
- **Area types (Stub, Totally Stubby, NSSA):** areas that don't need full external routes — they receive a default route instead, saving CPU and memory. Remember the loopback *stub host*? Here the concept is different — we're talking about an OSPF area type, not an isolated host.
- **Authentication:** *clear-text* or encryption (MD5, HMAC-SHA) to prevent unauthorized routers from joining the domain.

### Applying the second layer of refinement

Throughout this material, we've seen that the Neighbor ID matches the IP address of the interface where OSPF was activated. The truth is that OSPF doesn't care how creative your device's hostname is. The protocol doesn't understand names — it identifies each device through a Router ID (RID), a 32-bit identifier (with the same format as an IP address) that must be absolutely unique within the OSPF domain.

To assign this identifier, the OSPF process follows a specific election hierarchy:

1. **Manual configuration:** if the admin goes to the CLI and sets the router-id manually, the protocol obeys immediately and ignores everything else.
2. **Highest IP address on a Loopback interface:** if no manual config exists, OSPF checks all active Loopbacks and picks the one with the highest IP.
3. **Highest IP address on a Physical interface:** if the previous options fail, it looks for the active physical interface with the highest IP. Warning: this is not recommended! If that interface goes down, the process becomes unstable.

Manually configuring the RID prevents headaches down the road and ensures stability. Below are the commands applied on RT1 and RT2 to manually set the Router ID:

```cisco
RT1(config)# router ospf 1
RT1(config-router)# router-id 1.1.1.1
```

```cisco
RT2(config)# router ospf 2
RT2(config-router)# router-id 2.2.2.2
```

## Conclusion: The End of the Beginning

We've finally made it to the end of this first chapter. As we've seen, OSPF isn't just some protocol that blasts routes out randomly. It's methodical — it practically runs a background check with Hello packets (the Tinder of routers, if you will) and builds a complete network map before making any decisions, using Dijkstra's algorithm.

I know it sounds repetitive to say what everyone says, but understanding these theoretical foundations — from the rules for forming neighbors, through the state machine, all the way to cost calculation — is exactly what separates a mere CLI monkey from a true Network Engineer. I can tell you from experience: when something goes wrong, your fundamentals are what will save you.

In this Part 1 we've already gotten our hands dirty with labs, packet captures, *traceroute*, configs, and those satisfying logs flooding the screen until we hit FULL. But honestly? There's still so much more ground to cover. Theory and practice need to go hand in hand — like Lennon and McCartney, like Tweety and Sylvester...

In Part 2 of this series, we'll level up: Hello packets, LSA types, more complex topologies to illustrate the concepts, stub/NSSA areas, summarization, real troubleshooting, and scenarios closer to day-to-day work. In the troubleshooting material, we'll also demonstrate in practice the *auto-cost reference-bandwidth* adjustment to fix the equal-cost issue between FastEthernet and GigabitEthernet, and the precedence behavior when there's a conflict between the *network statement* and the *ip ospf* configured directly on the interface. I hope the foundation we built here — adjacency, cost, SPF — serves you well in the near future.

So fire up your favorite emulator (Packet Tracer, GNS3, EVE-NG, PNETLab) and I'll see you in the next article. Just a reminder — I use PNETLab, and the files are fully compatible with EVE-NG, and they'll come with the interfaces pre-configured!

Let's master OSPF together — LET'S GOOOOO!
