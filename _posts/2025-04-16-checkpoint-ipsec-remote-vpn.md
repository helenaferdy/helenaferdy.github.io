---
title: Check Point IPSec Remote Access VPN
date: 2025-04-18 07:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point IPsec Remote Access VPN allows users to securely connect to their organization’s internal network over the internet using an encrypted IPsec tunnel. It provides authenticated access to resources as if the user were on-site, ensuring data confidentiality and integrity.

<br>

## IPSec VPN 

On Check Point SmartConsole, here we enable the IPSec VPN blade

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/01.png)

<br>

On IPSec VPN, include this gateway into the RemoteAccess VPN Community

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/02.png)

<br>

Then on VPN Clients >> Remote Access, enable Visitor Mode

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/03.png)

<br>

Next on Office Mode, select Allow Office Mode and assign the VPN IP Pool

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/04.png)

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/05.png)

<br>

## VPN User Group 

Next configure the VPN User Group, which in this case includes one user named "vpnuser"

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/06.png)

<br>

Set that User Group as the VPN Community's Participant so we can use it to login to VPN

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/07.png)

<br>

Finally, we just need to make a policy to allow VPN users to access internal networks

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/08.png)

<br>

## Connecting to VPN

Now we can try connecting to the VPN gateway

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/09.png)

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/10.png)

<br>

Here we are getting the correct assigned IP Address from VPN IP Pool, we also get the routes to internal network

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/11.png)

<br>

The logs show the successfull vpn connections

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/12.png)

<br>

## Split Tunnel

By default Split Tunneling is on, but we can modify what segments to advertise to VPN clients, first we disable the "Allow Clients to route traffic to this gateway"

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/13.png)

<br>

Then on VPN Domain, we can specify the segments to advertise

![x](/static/2025-04-16-checkpoint-ipsec-remote-vpn/14.png)

<br>








