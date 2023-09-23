---
title: Site to Site VPN between Palo Alto and Fortigate
date: 2023-09-23 07:30:00 +0700
categories: [Security, Palo Alto]
categories: [Security, Fortigate]
tags: [palo alto, fortigate, vpn]
---


<br>

## What is Site to Site VPN?

A site-to-site VPN is a network setup that securely connects multiple remote locations or offices over the internet, creating a private and encrypted communication link between them. It allows different sites to communicate as if they were on the same local network.

This site-to-site VPN will use IPsec to secure the communication between remote locations. IPsec provides encryption and authentication to ensure the confidentiality and integrity of data exchanged

<br>
<br>

## Network Topology

Here's the topology for this Site-to-site VPN setup

![x](/static/2023-09-23-palo-forti-vpn/00.png)

<br>
<br>

## Configuring VPN on Fortigate

On Fortigate, go to VPN >> IPsec Wizard, create new and choose custom

![x](/static/2023-09-23-palo-forti-vpn/01.png)

<br>

Give it a name, and set the remote gateway and the outside interface

![x](/static/2023-09-23-palo-forti-vpn/02.png)

<br>

On authentication, select Pre-shared Key and type in the key, and on Phase 1 Proposal, fill in the Encryption and Authentication

![x](/static/2023-09-23-palo-forti-vpn/03.png)

<br>

On Phase 2, fill in the Local Subnet and the Remote Subnet, and also fill in the Encryption and Authentication

![x](/static/2023-09-23-palo-forti-vpn/04.png)

<br>

Save it, and this is what we end up with

![x](/static/2023-09-23-palo-forti-vpn/05.png)

<br>

Next, add a static route going to the remote subnet going out on the VPN interface

![x](/static/2023-09-23-palo-forti-vpn/06.png)

<br>

Then create new policy for traffic going out from Inside to VPN

![x](/static/2023-09-23-palo-forti-vpn/07.png)

<br>

And another policy for traffic coming in from VPN to Inside

![x](/static/2023-09-23-palo-forti-vpn/08.png)

<br>

This is the policies we end up with

![x](/static/2023-09-23-palo-forti-vpn/09.png)

<br>

And that should be all the configuration needed on Fortigate side


<br>
<br>

## Configuring VPN on Palo Alto

First, create a new Tunnel Interface

![x](/static/2023-09-23-palo-forti-vpn/09z.png)

<br>

Then we mirror the Fortigate's Phase 1 configuration on On Network >> IKE Crypto

![x](/static/2023-09-23-palo-forti-vpn/10.png)

<br>

Then we also mirror the Phase 2 configuration on IPsec Crypto

![x](/static/2023-09-23-palo-forti-vpn/11.png)

<br>

Next create a IKE Gateway, configure the local and remote interface and supply the Pre-Shared Key

![x](/static/2023-09-23-palo-forti-vpn/12.png)

<br>

And on IKE Crypto Profile, select the one created earlier 

![x](/static/2023-09-23-palo-forti-vpn/12a.png)

<br>

Next create an IPsec Tunnel, select the Tunnel Interface, IKE Gateway and IPsec Crypto Profile

![x](/static/2023-09-23-palo-forti-vpn/13.png)

<br>

And on Proxy ID, supply the local and remote subnet

![x](/static/2023-09-23-palo-forti-vpn/13a.png)

<br>

Then add a static route to remote subnet going through Tunnel Interface

![x](/static/2023-09-23-palo-forti-vpn/14.png)

<br>

Lastly, make sure there's a Policy to allow connection both way between VPN and Inside Zone

![x](/static/2023-09-23-palo-forti-vpn/16.png)

<br>

Commit the changes.

<br>
<br>

## Testing the Connection

Now connecting from the inside network of fortigate to the inside network of palo alto should succeed

![x](/static/2023-09-23-palo-forti-vpn/17.png)

<br>

The same with the other way around

![x](/static/2023-09-23-palo-forti-vpn/18.png)

<br>