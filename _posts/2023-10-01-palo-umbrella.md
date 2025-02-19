---
title: Palo Alto - SIG Tunnel to Cisco Umbrella
date: 2023-10-01 17:30:00 +0700
categories: [Security, Palo Alto]
tags: [Palo Alto, Umbrella]
---

<br>

A Secure Internet Gateway (SIG) is a cloud-based service that provides secure and filtered access to the internet for users and devices within an organization. <br>
Cisco Umbrella SIG uses tunnels to establish secure connections between Palo Alto's network and the cloud-based Umbrella service. This tunnel encrypts the traffic between Palo Alto and Umbrella service, ensuring the confidentiality and integrity of data as it travels over the internet.

![x](/static/2023-10-01-palo-umbrella/00.png)

<br>
<br>

## Deployment Topology

On this deployment, we will establish a Site-to-site VPN IPsec Tunnel connection between the Palo Alto and the Umbrella Cloud, making all the internet traffic coming form the Inside Segment will be routed to the Umbrella first before heading to the Internet

![x](/static/2023-10-01-palo-umbrella/00a.png)

<br>
<br>

## Configuring Network Tunnel on Umbrella

On Cisco Umbrella, go to Deployments >> Network Tunnels, add new for Palo Alto

![x](/static/2023-10-01-palo-umbrella/01.png)

<br>

Here's after the tunnel added, the entry is present with status of Unestablished

![x](/static/2023-10-01-palo-umbrella/02.png)

<br>
<br>

## Configuring Network Tunnel on Palo Alto

Cisco provides a decent deployment guide [here](https://docs.umbrella.com/umbrella-user-guide/docs/manual-palo-alto-ipsec-deployment) to configure tunnel on Palo Alto

![x](/static/2023-10-01-palo-umbrella/02a.png)

<br>

First creata a new Tunnel Interface

![x](/static/2023-10-01-palo-umbrella/03.png)

<br>

Next create a IKE Crypto Profile

![x](/static/2023-10-01-palo-umbrella/04.png)

> An IKE (Internet Key Exchange) Crypto Profile is a configuration that defines the parameters and settings used for the initial phase of establishing a VPN tunnel. It includes the encryption and authentication methods to be used during the key exchange process. 

<br>

Next create an IPSec Crypto Profile

![x](/static/2023-10-01-palo-umbrella/05.png)

> An IPSec (Internet Protocol Security) Crypto Profile is a configuration that defines the parameters and settings used for the second phase of establishing a VPN tunnel, which involves securing the data traffic between two sites. It specifies the encryption and authentication methods for protecting the actual data transmitted over the VPN tunnel.

<br>

After that create an IKE Gateway

![x](/static/2023-10-01-palo-umbrella/06.png)

![x](/static/2023-10-01-palo-umbrella/06a.png)

> An IKE (Internet Key Exchange) Gateway is a configuration that defines the properties of a remote peer or site with which the firewall will establish a VPN tunnel. It includes information such as the remote peer's IP address, pre-shared key, and the specific IKE Crypto Profile to be used for key exchange.

<br>

Finally create the IPSec Tunnel

![x](/static/2023-10-01-palo-umbrella/07.png)

> IPSec tunnel is the actual secure communication channel established between the Palo Alto and remote device (Umbrella), using the profiles configured earlier

<br>


Commit the changes.

<br>
<br>

## Verfying the Tunnel Connection

After commited, now the IPSec tunnel status on Palo Alto should be Up

![x](/static/2023-10-01-palo-umbrella/08.png)

<br>

Over on the other end on Umbrella, the Network Tunnel can also be seen Up and Active

![x](/static/2023-10-01-palo-umbrella/09.png)

<br>

Lastly before testing the traffic, create a Policy Based Forwarding (Policy-based Routing/PBR) to send traffic from client's segment to the tunnel

![x](/static/2023-10-01-palo-umbrella/10.png)

<br>

And now all internet traffic will be proxied by Umbrella, acting as the Secure Internet Gateway

![x](/static/2023-10-01-palo-umbrella/11.png)

Here we can see the reports of the traffic coming from the palo's internal segment is inspected before being forwarded to the actual destination on the internet

<br>




