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

Hey, how's it going? This was supposed to be one single article covering a topic from start to finish. But as I kept writing, it just kept growing! I have a very hard time knowing when to stop. Halfway through, I decided to split it into parts so it wouldn't be too long and tiring to read. I chose this name because I'm a big fan of a Brazilian tech platform called [*LinuxTips*](https://linuxtips.io/), which uses this naming style for their courses. This article covers a lot of information, but I also highly recommend the course [*Descomplicando o OSPF*](https://gustavokalau.com.br/) by Professor Gustavo Kalau (note: the course is in Portuguese!) if you want to learn even more.

Before we talk about OSPF, let's review what a dynamic routing protocol actually does. These concepts are simple, but they are fundamental. They will prepare us for everything else we will learn.

The main functions of a routing protocol include:

- **Neighbor and network discovery:** Finding other routers on the same network automatically. This means you don't have to add routes manually every time the network grows.
- **Best path calculation and selection:** Using math and logic to look at the network (speed, delay, number of routers in the way) and choose the best route for each destination.
- **Loop prevention:** Using mechanisms that stop packets from traveling in circles forever. In link-state protocols like OSPF, all routers use a map to build routes with no loops.
- **Fault tolerance:** Reacting quickly when a cable breaks or a device goes offline by finding alternative routes.
- **Classless routing:** Supporting modern IP addressing (VLSM and CIDR) by including subnet masks when sharing routes.

## Core Concepts

Let's talk about OSPFv2. OSPFv2 (Open Shortest Path First version 2) is an internal dynamic routing protocol (IGP). It is a link-state protocol, standardized by the IETF in RFC 2328. To understand what this means, let's compare it to older protocols like RIP (which is a distance vector protocol):

- **Distance vector:** This type of routing uses the number of "hops" (routers it passes through) to choose the best path. Fewer hops = better path. It is like driving by only looking at distance signs. You trust the numbers, but you don't know if there is traffic or if the road is bad. In real networks, cables have different speeds. So, the path with fewer hops might actually be the slowest one. Surprisingly, the "best" path can become the worst path.
- **Link-state:** Routers share information like puzzle pieces. In OSPF, these messages are called LSAs (Link-State Advertisements). These pieces are put together in a database called the LSDB (Link-State Database). Once the puzzle is complete, each router builds an exact and complete map of the network. This map includes all routers, connections, link status (Up/Down), and the "cost" of each path.

The OSPF lifecycle basically has three phases::

1. Finding neighbors and making connections (adjacencies).
2. Building and sharing the LSDB.
3. Running the SPF (Dijkstra) algorithm to find the best paths.

### SPF and Metric

With the network map (LSDB) ready, OSPF finds the best path using the SPF (Shortest Path First) algorithm, created by Edsger W. Dijkstra. The calculation has four steps:

**1. Starting point:** Each router runs SPF independently, placing itself at the top (the root) of the Shortest Path Tree (SPT).

**2. Metric:** The rule for choosing the best path is the cost. The cost depends on the link's speed (bandwidth). Faster links have a lower cost.

![OSPF Cost Formula](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-11.png)

OSPF looks at the path, but pay close attention to this important detail: it only uses bandwidth (speed) to calculate the cost.
Another key point: by default, Cisco routers use a reference bandwidth of 100 Mbps. Because of this, both FastEthernet (100 Mbps) and GigabitEthernet (1000 Mbps) ports get the same cost of 1. This can cause problems! To fix it, you should use the **auto-cost reference-bandwidth** command on all routers in your OSPF network. We will show this in the troubleshooting article. Just as a note, another protocol called EIGRP can look at speed, delay, load, and reliability.

**3. Cumulative cost:** The router adds up the cost of each outgoing interface along the path to the destination.

**4. RIB installation:** The path with the lowest total cost wins. The losing path stays in the LSDB as a backup. The winning path goes into the routing table (RIB). If there is a tie, OSPF uses both paths and splits the traffic (this is called load-balancing or ECMP).

To show the difference between these two types of protocols, let's look at the topology below:

![RIP and OSPF Topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1.png)

Let's see how a Distance Vector protocol like RIP works:

- As a starting point, let's say the blue connections are fast Ethernet Links and the yellow ones are slow Serial Links.
- Now imagine a packet needs to travel from router RT2 to RT7.
- Based on our topology, the shortest path (fewest routers) would be RT2 > RT4 > RT7.
- However, because the serial links are slower, this "shortest path" might not actually be the fastest way to reach the destination!

![RIP path in the topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-1.png)

Now let's see how OSPF works using link-state logic. Using the cost formula, we can calculate the costs as shown in the image below:

![OSPF costs in the topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-2.png)

Now that OSPF has calculated the total cost, let's see how the packets actually travel!
OSPF is already turned on in this topology. We will explain how to turn it on later. You can click on [*download*](https://drive.google.com/drive/folders/1PFSQtfXWONkJiUJ5dYLB7eKL-c4M5bcl?usp=drive_link) to get the lab files. You can practice as much as you want! I use PNETLab, and the files work perfectly with EVE-NG too.

When we look at the routing table, we see the route to RT7 has a cost of 8. You might ask: "Why 8? If we add the costs of the outgoing interfaces, it equals 7!" Here is a small detail that can confuse you. According to OSPF rules (and Cisco routers), a loopback interface is treated as a stub host. This means it automatically gets a fixed cost of 1. A stub in networking is like a "dead-end" street. Traffic can go in and come out, but it doesn't pass through to another place. Do not confuse this with a stub area (we will talk about that in future articles).

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

Below we can use the *traceroute* command to see the actual path the packets travel:

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

Finding a neighbor (Phase 1) is just the first step. Before routers start sharing information, they need to agree on some basic rules. They use Hello packets to check a list of requirements. If anything is different, they will not become neighbors!

For the connection process to begin, the following settings must be exactly the same on both sides of the cable:

- **Area ID:** If one router is in Area 0 and the other is in Area 1 on the same connection, they will not move forward.
- **Subnet and Mask:** The IP addresses of the connected interfaces must be in the exact same network.
- **Hello and Dead Timers:** The timers must match (usually 10 seconds for Hello and 40 seconds for Dead).
- **Authentication:** If you use a password, it must be exactly the same on both sides.
- **Special Area Flags (Stub/NSSA):** Both routers must agree on what kind of area they are in.

There are also two more rules to make sure the process doesn't get stuck later:

- **Unique Router IDs:** Each router needs its own unique identity (RID). If two routers have the same RID, the network will break.
- **Matching MTU (Maximum Transmission Unit):** The maximum packet size must be the same. If one side is 1500 bytes and the other is 1400, they will start talking, but the connection will fail halfway. This happens because OSPF does not support breaking packets into smaller pieces (fragmentation).

> The OSPF process number does NOT need to be the same between routers to form an adjacency.

To show this better, we will use this topology:

![OSPF adjacency topology](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-6.png)

When everything is correct, the OSPF routers go through seven steps (states) to share their databases. They talk directly using the IP protocol (protocol ID 89). There are five types of packets: Hello, DBD, LSR, LSU, and LSAck. Here are the steps:

1. **Down:** The starting state. OSPF is active, but no Hello packet has arrived yet. The router sends a Hello to the multicast address 224.0.0.5 (All OSPF Routers) and waits for an answer.
2. **Init:** A Hello arrived from a neighbor, but it is only one-way communication. The router does not see its own ID in the neighbor's packet.
3. **2-Way:** The router sees its own ID in the neighbor's Hello packet. Two-way communication is confirmed! Now, OSPF checks the interface's Network Type to decide what to do next. We'll focus on the two most common types here. Point-to-point (a direct connection between two routers), it skips the DR/BDR election and moves to the next step. Broadcast (the default for Ethernet), because there can be many routers on the same switch, OSPF elects a Boss (Designated Router or DR) and a Vice-Boss (Backup Designated Router or BDR). The other routers (DROTHERs) stay in the 2-Way state with each other. They only go to the Full state with the DR and BDR. This stops the network from getting too crowded.
4. **ExStart:** The basic connection is ready. Now the routers elect a Master and a Slave (the router with the highest ID wins). This decides who starts sending DBD (Database Description) packets.
5. **Exchange:** The routers swap DBD packets. Think of DBDs as the "index" or "menu" of the database. They compare what they have, but they don't send the full details yet.
6. **Loading:** The router asks for the missing details using an LSR (Link-State Request). The neighbor answers with an LSU (Link-State Update). Finally, the router confirms it received the update with an LSAck (Link-State Acknowledgment).
7. **Full:** The databases are exactly the same. LET'S GOOOOO! The router runs the SPF algorithm and puts the best routes in the routing table (RIB).

> Note: Phases 2 and 3 happen in the **Exchange**, **Loading**, and **Full** states. The LSDB is shared. Then, each router runs SPF independently to build the RIB.

### Enabling OSPF on Cisco IOS

We've covered a lot of theory, but hands-on practice is essential for real understanding. Before any Hello packet leaves the router, we must turn on OSPF. In Cisco routers, there are two ways to do this:

#### 1. Network statement (global configuration under *router ospf*)

This is the classic way. You enter the OSPF configuration and choose which networks (and interfaces) will run OSPF. This configuration was done on router RT1:

```cisco
router ospf 1
 network 10.0.0.1 0.0.0.0 area 0
```

Many people think the network statement command advertises a route, but it works a bit differently. Actually, it works like an interface selector. Here is how it works:

- You provide an IP address and a wildcard mask.
- The router checks its interfaces.
- If the interface's IP address matches the wildcard, OSPF becomes active on it.

That's it!

In the example of IP address 10.0.0.1 with wildcard mask 0.0.0.0, we're being very specific. Only the interface with that exact IP will participate in OSPF. If the router doesn't find that IP, OSPF doesn't turn on, and no route is advertised. Understanding this will help you fix network problems faster.

#### 2. Interface-level configuration

The other way is to turn on OSPF directly on the interface, as we did on router RT2:

```cisco
interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip ospf 2 area 0
```

Both ways turn on OSPF. The only difference is where you type the command: globally or directly on the interface.

> On Cisco IOS, specific commands always win over general commands. If there is a conflict (for example, the global config says Area 0, but the interface config says Area 1), the interface config wins! We will show this in the troubleshooting article.

##### Configuration Tip - Network Type

Remember that Ethernet networks behave like a broadcast network by default. Since our routers are connected directly to each other, we should change the network type to point-to-point. This stops the router from doing the unnecessary DR/BDR election. We will see this in action shortly.

#### Back to the main topic

Now let's watch the routers become neighbors:

- After we configure OSPF, the router starts sending Hello packets. You can see this in Wireshark:

![OSPF Hello packet capture](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-8.png)

![OSPF Hello packet detail](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-9.png)

![OSPF Hello packet in capture](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-10.png)

We used the **show ip protocols** command on RT1 to see the active routing protocols:

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

Now let's check the *show ip protocols* output on router RT2:

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

We can see the difference in how OSPF was configured by looking closely at the **show ip protocols** output:

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

If we turn on *debug ip ospf adj*, we can see the states changing step by step on **RT1**:

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

- **Areas:** We divide the network into areas to isolate problems and save CPU power. Area 0 is the backbone. All other areas must connect to it to communicate.
- **Router ID:** A unique 32-bit name for the router. It looks like an IP address, but it is just an ID.
- **DR and BDR:** On shared networks (like Ethernet switches), OSPF elects a Boss (DR) and a Vice-Boss (BDR) to manage updates.
- **Summarization:** Combining multiple routes into one smaller route to keep the database stable.
- **Area types (Stub, Totally Stubby, NSSA):** Special areas that do not receive all external routes. Instead, they receive a default route. This saves memory. (Remember the loopback stub host? This is different. This is an area, not a single interface).
- **Authentication:** Using passwords (plain text or encrypted) so only trusted routers can join the network.

### Configuration Tip - The Router ID

As we saw, the Neighbor ID is usually the IP address of the interface. OSPF does not care about the router's name (hostname). It identifies routers using the Router ID (RID).

To choose this ID, OSPF follows three rules in order:

1. **Manual configuration:** If you set the router-id manually, OSPF will use it as its first option. This is the best way.
2. **Highest IP address on a Loopback interface:** If there is no manual ID, OSPF looks at the active Loopback interfaces and picks the highest IP address.
3. **Highest IP address on a Physical interface:** If there are no Loopbacks, it picks the highest IP address on an active physical port. Warning: this is not recommended! If that port goes offline, the OSPF process can become unstable.

Setting the RID manually prevents problems and keeps the network stable. Here is how we did it on RT1 and RT2:

```cisco
RT1(config)# router ospf 1
RT1(config-router)# router-id 1.1.1.1
```

```cisco
RT2(config)# router ospf 2
RT2(config-router)# router-id 2.2.2.2
```

## Conclusion: The End of the Beginning

We have finally reached the end of this first chapter! As we saw, OSPF does not just send routes randomly. It is very organized. It checks the neighbor carefully with Hello packets (like a dating app for routers!) and builds a complete network map using Dijkstra's algorithm before making any decisions.

It sounds repetitive, but understanding these basics, how neighbors form, the state machine, and cost calculations, is what makes you a true Network Engineer, not just someone who copies commands. When the network breaks, these basic concepts will save you.

In Part 1, we practiced with labs, packet captures, traceroute, configurations, and those satisfying logs that show FULL. But there is still much more to learn! Theory and practice go perfectly together, like Lennon and McCartney, or SGA and the referees.

In Part 2, we will go deeper. We will look at Hello packets, LSA types, more complex networks, stub areas, and real troubleshooting. We will also test the auto-cost reference-bandwidth command and see what happens when global and interface configurations conflict. I hope these fundamentals serve you well!

So, open your favorite emulator (Packet Tracer, GNS3, EVE-NG, or PNETLab), and I will see you in the next article. Remember, you can download my lab files to practice, and to make things easier, the interfaces come pre-configured!

Let's master OSPF together — LET'S GOOOOO!
