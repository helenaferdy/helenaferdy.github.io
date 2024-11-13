---
title: Cisco ASA Multi Context
date: 2024-11-12 09:00:00 +0700
categories: [Security, Cisco Adaptive Security Appliance (ASA)]
tags: [ASA]
---

Cisco ASA multi-context mode allows a single physical ASA device to be partitioned into multiple virtual firewalls, each acting as a separate, isolated instance with its own security policies, configurations, and interfaces. With a *shared interface*, these virtual contexts can use the same physical interface for network traffic, allowing for resource-efficient management of traffic while keeping each context's traffic logically separated based on security policies and VLAN tagging.

## ASA Multi Context

Here we have an ASA that works as a onearm-transit firewall for two differen networks. To enable Multi Context run command "mode multiple"

![x](/static/2024-11-12-asa-multicontext/01.png)

<br>

After a reboot, run "show mode" and "show context" to verify Multi Context configuration

![x](/static/2024-11-12-asa-multicontext/02.png)

> By default, we have the 2 predefined contexts, **system context** and **admin context** <br>
> * System Context : This is the top-level management context that holds the global configuration for the entire ASA device in multi-context mode. It is responsible for creating and managing contexts, allocating resources, and handling any device-wide settings, like interface allocations and global routing configurations. The system context does not handle user traffic or have specific firewall policies. <br>
> * Admin Context : The admin context is a designated user-defined context that acts as the main context for administrative tasks within the ASA. It is created by default when the device is set to multi-context mode and is where most configuration for user traffic and firewall policies are managed. The admin context can also be configured with unique security policies, making it both a functional firewall context and the main context for system-wide management tasks.

<br>

Next lets create a VLAN interfaces which we will allocate to each context

![x](/static/2024-11-12-asa-multicontext/03.png)

```shell
conf t
 int eth0
 no shut

 int eth0.21
 vlan 21

 int eth0.22
 vlan 22
```

<br>

After that we create the 2 contexts, C1 and C2 and we allocate the respective interface

![x](/static/2024-11-12-asa-multicontext/04.png)

```shell
context C1
 description Context1
 allocate-interface e0.21 int21 visible
 config-url disk0:/C1.cfg

context C2
 description Context2
 allocate-interface e0.22 int22 visible
 config-url disk0:/C2.cfg
```

<br>

### Context C1

Now we can get into the context with command "changeto context", and we can then proceed configuring the context as a usual firewall

![x](/static/2024-11-12-asa-multicontext/05.png)

```shell
changeto context C1

conf t
 int int21
 ip add 172.100.0.2 255.255.255.0
 nameif TRANSIT
```

<br>

Next we configure the firewall policy for traffic going through this transit firewall

![x](/static/2024-11-12-asa-multicontext/06.png)

```shell
object-group network CLIENT
 network-object host 10.100.0.10

object-group network SERVER
 network-object host 100.100.100.100

access-list ACL-TRANSIT extended permit ip object-group CLIENT object-group SERVER
access-list ACL-TRANSIT extended permit ip object-group SERVER object-group CLIENT

access-group ACL-TRANSIT in interface TRANSIT
```

<br>

Finally lets add a default route to route all traffic back to the originating interface 

![x](/static/2024-11-12-asa-multicontext/07.png)

```shell
route TRANSIT 0 0.0.0.0 172.100.0.1
same-security-traffic permit intra-interface
```

<br>

And now we have connectivity from Client to Server thats going through the C1 Context Firewall

![x](/static/2024-11-12-asa-multicontext/08.png)

<br>

Looking at the access-lists, we can see the hitcount is going up

![x](/static/2024-11-12-asa-multicontext/09.png)

<br>

### Context C2

For Context C2, its the same drill for the other network. Here we just make an any-any firewall rule for simplicity

![x](/static/2024-11-12-asa-multicontext/10.png)

<br>

And now we also have connectivity on the second network going through the C2 Context Firewall

![x](/static/2024-11-12-asa-multicontext/11.png)

![x](/static/2024-11-12-asa-multicontext/12.png)

<br>
<br>

## ASA Multi Context High-Availability

Before configuring failover, lets configure each interface wihin the contexts to have standby ip addresses

![x](/static/2024-11-12-asa-multicontext/13.png)

```shell
changeto context C1
 conf t
  int int21
   ip address 172.100.0.2 255.255.255.0 standby 172.100.0.3

changeto context C2
 conf t
  int int22
   ip address 172.200.0.2 255.255.255.0 standby 172.200.0.3
```

<br>

Next configure the failover on ASA1

![x](/static/2024-11-12-asa-multicontext/14.png)

```shell
failover lan unit primary
failover lan interface FO eth1
failover interface ip FO 1.1.1.1 255.255.255.252 standby 1.1.1.2

failover group 1
 primary
 preempt
failover group 2
 secondary
 preempt

context C1
 join-failover-group 1
context C2
 join-failover-group 2
```

<br>

Then on the ASA2, make sure its already on Multi Context Mode with no configuration at all, after that we can configure the failover

![x](/static/2024-11-12-asa-multicontext/15.png)

```shell
mode multiple

failover lan unit secondary
failover lan interface FO eth1
failover interface ip FO 1.1.1.1 255.255.255.252 standby 1.1.1.2
```

<br>

Running command "failover" on both nodes will start the failover process, and running "show failover" shows the failover status where Group 1 has ASA1 as the active whereas Group 2 has ASA2

![x](/static/2024-11-12-asa-multicontext/16.png)

<br>

On the ASA2, we can see all configuration including the contexts are copied over

![x](/static/2024-11-12-asa-multicontext/17.png)

<br>

Logging into each context shows that each has different active-standby configuration, but here the interfaces are not yet monitored

![x](/static/2024-11-12-asa-multicontext/18.png)

<br>

Login to each context and run "monitor interface TRANSIT" to monitor the interface

![x](/static/2024-11-12-asa-multicontext/19.png)

<br>

### Failing Over

If any interface is down on one node, the other node will take owenership of the context

![x](/static/2024-11-12-asa-multicontext/20.png)

<br>

Same goes if the entire node is down

![x](/static/2024-11-12-asa-multicontext/21.png)

<br>
<br>
<br>
<br>
<br>

Anyway here's the PBR configuration on the switch side

![x](/static/2024-11-12-asa-multicontext/22.png)

```shell
##SW1
ip access-list extended ASA-PBR
 permit ip host 10.100.0.10 host 100.100.100.100

route-map ASA-RM permit 10
 match ip address ASA-PBR
 set ip next-hop 172.100.0.2
        
interface Vlan100
 ip address 10.100.0.1 255.255.255.0
 ip policy route-map ASA-RM
```

<br>










