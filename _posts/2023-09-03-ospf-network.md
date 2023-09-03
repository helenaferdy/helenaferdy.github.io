---
title: OSPF Network Types
date: 2023-09-01 10:30:00 +0700
categories: [Networking, OSPF]
tags: [OSPF]
---

<br>

## OSPF Network Types

In OSPF, there are 5 different types of connections between routers:

<br>

![x](/static/2023-09-02-ospf-network/01.png)

> 1. Point-to-Point (P2P): <br>
In a Point-to-Point connection, OSPF is configured between two directly connected routers. This type of connection is typically used in scenarios where there is a single link between two routers. P2P connections don't require the use of DR/BDR (Designated Router/Backup Designated Router) election.

<br>

![x](/static/2023-09-02-ospf-network/02.png)

> 2. Broadcast: <br>
In a Broadcast network type, multiple routers are connected to a same network segment. In this type of network, OSPF routers elect a DR and a BDR to reduce the amount of OSPF traffic and improve efficiency.

> 3. Non-Broadcast Multi-Access (NBMA): <br>
NBMA networks are similar to Broadcast networks but lack the automatic broadcast capabilities. In NBMA networks, OSPF routers can't rely on automatic DR/BDR election due to the lack of broadcast, so the neighborship adjacency needs to be configured manually

<br>

![x](/static/2023-09-02-ospf-network/03.png)

> 4. Point-to-Multipoint (P2MP): <br> 
In a Point-to-Multipoint connection, a router is connected to multiple remote routers as if it were a point-to-point link with each of them, but are still in the same segment. P2MP connections simplifies OSPF configuration and doesn't require DR/BDR election.

> 5. Point-to-Multipoint Non-Broadcast (P2MNB): <br> 
Similar to P2MP, P2MNB connections are used in a network with no broadcast connectivity. Neighborship adjacency is configured manually.


<br>
<br>

## Point-to-Point (P2P)

![x](/static/2023-09-02-ospf-network/01.png)

Here's the configuration for point-to-point connection between XE1 and XE2

> XE1

```shell
interface GigabitEthernet2
 ip address 99.0.0.1 255.255.255.0
 ip ospf network point-to-point
 ip ospf 1 area 0
```

> XE2

```shell
interface GigabitEthernet2
 ip address 99.0.0.2 255.255.255.0
 ip ospf network point-to-point
 ip ospf 1 area 0
```

<br>

And if we check the neighbor status, they should discover each other

![x](/static/2023-09-02-ospf-network/04.png)

<br>

Checking the OSPF type, it shows that it is indeed a point-to-point connection

![x](/static/2023-09-02-ospf-network/05.png)


<br>
<br>

## Broadcast

![x](/static/2023-09-02-ospf-network/02.png)

Here's the configuration for broadcast connection between XE1, XE2 and XE3

> XE1

```shell
interface GigabitEthernet2
 ip address 99.0.0.1 255.255.255.0
 ip ospf network broadcast
 ip ospf 1 area 0
```

> XE2

```shell
interface GigabitEthernet2
 ip address 99.0.0.2 255.255.255.0
 ip ospf network broadcast
 ip ospf 1 area 0
```

> XE3

```shell
interface GigabitEthernet2
 ip address 99.0.0.3 255.255.255.0
 ip ospf network broadcast
 ip ospf 1 area 0
```

<br>

If we take a look at the neighborship adjacency status, we can see all 3 routers have discovered each other, with an additional information about the DR/BDR roles

![x](/static/2023-09-02-ospf-network/06.png)

Here XE3 is elected as the DR because it has the highest router-id number, followed by XE2 as the BDR, while XE1 remains in DROTHER status.

> DR (Designated Router): DR is responsible for performing LSA flooding tasks to the entire segment. <br>
> BDR (Backup Designated Router): The BDR is the router that takes over as the DR if the current DR fails. <br>
> DROther (DR Other Router): DROther routers are OSPF routers that neither hold the DR nor the BDR role. <br>

<br>

And if we check the OSPF connection type, it'll show broadcast.

![x](/static/2023-09-02-ospf-network/07.png)

<br>
<br>

## Point-to-Multipoint (P2MP)

![x](/static/2023-09-02-ospf-network/03.png)

Here's the configuration for P2MP connection between XE1, XE2, XE3 and XE4

> XE1

```shell
interface GigabitEthernet2
 ip address 99.0.0.1 255.255.255.0
 ip ospf network point-to-multipoint
 ip ospf 1 area 0
```

> XE2

```shell
interface GigabitEthernet2
 ip address 99.0.0.2 255.255.255.0
 ip ospf network point-to-multipoint
 ip ospf 1 area 0
```

> XE3

```shell
interface GigabitEthernet2
 ip address 99.0.0.3 255.255.255.0
 ip ospf network point-to-multipoint
 ip ospf 1 area 0
```

> XE4

```shell
interface GigabitEthernet2
 ip address 99.0.0.4 255.255.255.0
 ip ospf network point-to-multipoint
 ip ospf 1 area 0
```

<br>

Checking the neighbor status, we see all the routers have been discovered, but what different from broadcast is there's no DR/BDR election

![x](/static/2023-09-02-ospf-network/08.png)

<br>

And this also shows the type to be a pont-to-multipoint

![x](/static/2023-09-02-ospf-network/09.png)

<br>
<br>

## Non-Broadcast Multi-Access (NBMA) & Point-to-Multipoint Non-Broadcast (P2MNB)

For NBMA & P2MNB, the configuration is pretty much the same, but because the lack of broadcast/multicsat traffic, we have to manually build the neighborship adjacency with this command from one router side

> XE1

```shell
router ospf 1
 neighbor 99.0.0.2
 neighbor 99.0.0.3 
 neighbor 99.0.0.4 
```

<br>