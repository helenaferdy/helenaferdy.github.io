---
title: Fortigate Site-to-Site IPSec VPN
date: 2023-10-16 10:30:00 +0700
categories: [Security, Fortigate]
tags: [Fortigate, VPN]
---

<br>

A Site-to-Site IPSec VPN is a secure network connection that links two or more separate networks or sites over the internet with IPSec VPN Ecrypted Traffic

<br>
<br>

## Network Topology

Here's the topology for this deployment

![x](/static/2023-10-16-forti-s2s-vpn/00.png)

<br>
<br>

## Configuring VPN on Forti1

First of all, make sure both the nodes are abel to reach each other over WAN

![x](/static/2023-10-16-forti-s2s-vpn/01.png)

<br>

Then on Forti1 (node1), create a Site to Site VPN with IPSec Wizard

![x](/static/2023-10-16-forti-s2s-vpn/02.png)

<br>

Configure the remote IP Address of forti2 and set the PSK

![x](/static/2023-10-16-forti-s2s-vpn/03.png)

<br>

Then set the local and remote subnet to be advertised over VPN

![x](/static/2023-10-16-forti-s2s-vpn/04.png)

<br>

Review the configuration and hit Create

![x](/static/2023-10-16-forti-s2s-vpn/05.png)

<br>

After that, create 2 firewall policies to allow network coming out and in over VPN

![x](/static/2023-10-16-forti-s2s-vpn/05a.png)

<br>

And lastly, create a route going out to the VPN interface

![x](/static/2023-10-16-forti-s2s-vpn/05b.png)

<br>
<br>

## Configuring VPN on Forti2

On forti2, the configuration is pretty much the same but mirrored

![x](/static/2023-10-16-forti-s2s-vpn/06.png)

![x](/static/2023-10-16-forti-s2s-vpn/07.png)

![x](/static/2023-10-16-forti-s2s-vpn/08.png)

![x](/static/2023-10-16-forti-s2s-vpn/09.png)

![x](/static/2023-10-16-forti-s2s-vpn/09a.png)

![x](/static/2023-10-16-forti-s2s-vpn/09b.png)

<br>
<br>

## Testing the VPN Connections

Now we should have our IPSec VPN connetions to be in "Up" status

![x](/static/2023-10-16-forti-s2s-vpn/10.png)

![x](/static/2023-10-16-forti-s2s-vpn/11.png)

<br>

And we're able to ping from local subnet 1 to local subnet 2 

![x](/static/2023-10-16-forti-s2s-vpn/12.png)

<br>

Ping also works the other way around

![x](/static/2023-10-16-forti-s2s-vpn/13.png)

<br>

And looking at the details of the VPN, we can also see the traffic going through the VPN tunnel

![x](/static/2023-10-16-forti-s2s-vpn/14.png)

![x](/static/2023-10-16-forti-s2s-vpn/15.png)

<br>
























