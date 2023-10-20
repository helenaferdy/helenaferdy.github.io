---
title: Fortinet Hub-and-Spoke SD-WAN with ADVPN
date: 2023-10-17 11:30:00 +0700
categories: [SD-WAN, Fortinet SD-WAN]
tags: [SD-WAN, Fortigate, VPN]
---

<br>

## What Hub-and-Spoke SD-WAN with ADVPN?

Hub-and-Spoke SD-WAN with ADVPN is a setup where a central hub connects to multiple remote branch offices using SD-WAN technology. The use of ADVPN means that the VPN connections between the hub and spokes are established automatically and dynamically, simplifying network management and ensuring efficient and secure communication between all network locations.

> * Hub-and-Spoke: In this topology, there is a central hub that connects to multiple remote branch offices (spokes). The hub serves as a central point for network management and traffic distribution
> * ADVPN (Auto Discovery VPN): ADVPN is a feature in VPN that allows dynamic and automatic establishment of VPN connections between different network endpoints. Instead of configuring VPN tunnels manually for each remote site, ADVPN automatically discovers and establishes VPN connections as needed.

<br>
<br>

## Network Topology

Here's the network topology for this deployment, where we also have BGP as the overlay network to handle the routing of traffic between different SD-WAN edges

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/00.png)

<br>
<br>

## Configuring the Hub on Site 1

First of all, make sure all devices within the deployment are able to reach each other

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/01.png)

<br>

Next configure the IPSec VPN for WAN 1

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/02.png)

<br>

Configure the Remote Gateway to be Dialup User, disable Add Route, and enable Auto Discovery Sender

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/03.png)

> * Dialup User allows Spokes to dynamically form a Tunnel Connection to Hub, thus allowing multiple Spokes to establish IPSec VPN to Hub
> * Disable Add Route ensures IKE doesn't add routes when dynamic tunnel is negotiated, because routing will be handled by BGP
> * Auto Discovery Sender enabled means the Hub will periodically send out discovery messages to find compatible VPN peers so the Spokes can use it to automatically negotiate and establish VPN connections 

<br>

Configure the Pre-shared Key and Phase 1 Proposal as needed

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/04.png)

<br>

Same goes with the Phase 2 Proposal 

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/05.png)

<br>

Repeat the process for WAN 2 Interface and we end up with these 2 VPN Connections

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/06.png)

<br>

Next we configure the SD-WAN Zone having two members of the IPSec VPNs just created

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/07.png)

<br>

After that create two Policies to allow traffic from spokes to hub and between the spokes

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/08.png)

<br>

On the VPN 1 Interface, add an IP Address that'll be used by BGP

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/09.png)

<br>

Do the same for VPN 2

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/10.png)

<br>

Here's what we end up with on the Interface Configuration

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/11.png)

<br>

Lastly, configure BGP as the Overlay Routing Protocol for SD-WAN. <br>
set the Local AS to be 65001 and add a neighbor group containing the VPN 1 interface, make sure to enable the Route reflector client

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/13.png)

<br>

Same goes with the second VPN interface

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/14.png)

<br>

Next create a BGP Neighbor Range containing the network subnet of the VPN 1 interface

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/15.png)

<br>

Add another one for the VPN 2 interface

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/16.png)

<br>

And finally add the Local subnet, and hit save

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/17.png)

<br>
<br>

## Configuring the Spoke on Site 2

Create a IPSec VPN for interface WAN 1

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/20.png)

<br>

Configure the Remote Gateway going out to the Hub WAN 1 interface, disable Add Route and enable Auto Discovery Receiver

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/21.png)

> * When the Hub with Auto Discovery Sender sends out discovery messages to identify compatible peers, the Auto Discovery Receiver listens for these messages. When a compatible peer is found, the Auto Discovery Receiver can initiate the negotiation process for setting up a secure IPSec VPN tunnel between the two devices.

<br>

Configure the Pre-shared Key and Phase 1 Proposal as needed

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/22.png)

<br>

Same goes with the Phase 2 Proposal

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/23.png)

<br>

Repeat the process for the WAN 2 interface, and this is what we end up with

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/24.png)

<br>

Next, create an SD-WAN Zone containing these 2 VPNs as the members

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/25.png)

<br>

Create two Policies to allow traffic going in and out of the SD-WAN interface

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/26.png)

<br>

After that, configure the VPN Interfaces with IP Addresses to be used by BGP

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/27.png)

<br>

And configure the BGP with the same AS number of 65001, here we add a neighbor of the Hub WAN 1 interface

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/28.png)

<br>

Add the second neighbor for Hub's WAN 2 interface

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/28a.png)

<br>

Lastly add the Local subnet, and hit save

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/29.png)

<br>
<br>

## Configuring the Spoke on Site 3

On the Spoke on Site 3, the configuration is nearly identical with the Spoke on Site 2

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/30.png)

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/31.png)

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/32.png)

<br>
<br>

## Validating the Configuration

On Hub, we can see the IPSec VPNs have been automatically established to the spokes

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/40.png)

<br>

And on Routing Monitors, we can see all spokes' VPN interfaces as the BGP Neighbors

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/41.png)

<br>

And on BGP Paths, we can see the local subnets of the spokes being advertised by BGP

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/42.png)

<br>

On the client PC on Site 2, we can see the connectivity is up going to the Site 3

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/43.png)

<br>

Same goes with the other way around

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/44.png)

<br>

And on the Spokes, we can also see the VPN utilizations as the traffic flowing through the SD-WAN

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/45.png)

![x](/static/2023-10-17-forti-sdwan-hub-spoke-advpn/46.png)

<br>


