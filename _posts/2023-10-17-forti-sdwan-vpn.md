---
title: Fortinet SD-WAN with IPSec VPN
date: 2023-10-17 07:30:00 +0700
categories: [Software Defined Networking, Fortinet SD-WAN]
tags: [SD-WAN, Fortigate, VPN]
---

<br>

SD-WAN with IPSec VPN is a technology that combines SD-WAN and IPSec VPN to securely connect to a remote locations over WAN. SD-WAN optimizes network traffic and allows for centralized management, while IPSec VPN ensures data encryption and secure communication over the WAN.

<br>
<br>

## Network Topology

Here's the topology for this deployment

![x](/static/2023-10-17-forti-sdwan-vpn/00.png)

<br>
<br>

## Site to Site IPSec VPN Configuration

Here the Site to Site IPSec VPNs are already configured, the configuration can be seen in details [here](https://helenaferdy.github.io/posts/forti-s2s-vpn/)

> Site 1

![x](/static/2023-10-17-forti-sdwan-vpn/01.png)

> Site 2

![x](/static/2023-10-17-forti-sdwan-vpn/02.png)

<br>
<br>

## Configuring SD-WAN on Site 1

On Forigate on Site 1, create a new SD-WAN Zone

![x](/static/2023-10-17-forti-sdwan-vpn/03.png)

<br>

Then create a member of VPN 1 interface in the SD-WAN Zone

![x](/static/2023-10-17-forti-sdwan-vpn/04.png)

<br>

Do the same for VPN 2 interface

![x](/static/2023-10-17-forti-sdwan-vpn/05.png)

<br>

Here's the SD-WAN configuration we end up with

![x](/static/2023-10-17-forti-sdwan-vpn/06.png)

<br>

Next create a Firewall Policy to allow traffic going out to SD-WAN interface

![x](/static/2023-10-17-forti-sdwan-vpn/07.png)

<br>

And another Policy to allow traffic coming in

![x](/static/2023-10-17-forti-sdwan-vpn/08.png)

<br>

Lastly, add a route to go out using the SD-WAN interface

![x](/static/2023-10-17-forti-sdwan-vpn/09.png)

<br>
<br>

## Configuring SD-WAN on Site 2

For site 2, the configuration is exactly the same but mirrored

![x](/static/2023-10-17-forti-sdwan-vpn/10.png)

![x](/static/2023-10-17-forti-sdwan-vpn/11.png)

![x](/static/2023-10-17-forti-sdwan-vpn/12.png)

![x](/static/2023-10-17-forti-sdwan-vpn/13.png)

![x](/static/2023-10-17-forti-sdwan-vpn/14.png)

![x](/static/2023-10-17-forti-sdwan-vpn/15.png)

<br>
<br>

## Validating SD-WAN Configuration

Now both VPNs within the SD-WAN Zone should be up and running

![x](/static/2023-10-17-forti-sdwan-vpn/16.png)

![x](/static/2023-10-17-forti-sdwan-vpn/16a.png)

<br>

And we're able to ping from local subnet 1 to local subnet 2 

![x](/static/2023-10-17-forti-sdwan-vpn/17.png)

<br>

Ping also works the other way around

![x](/static/2023-10-17-forti-sdwan-vpn/18.png)

<br>

Now let's try shutting down the WAN connection for VPN1

![x](/static/2023-10-17-forti-sdwan-vpn/19.png)

<br>

SD-WAN will intelligently route the traffic to the up and running VPN 2

![x](/static/2023-10-17-forti-sdwan-vpn/20.png)

![x](/static/2023-10-17-forti-sdwan-vpn/21.png)

<br>


















