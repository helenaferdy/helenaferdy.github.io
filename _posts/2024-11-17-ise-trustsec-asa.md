---
title: Cisco ISE TrustSec with ASA
date: 2024-11-17 09:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE, RADIUS, ASA]
---

Cisco ISE TrustSec enables dynamic and scalable network segmentation by assigning Security Group Tags (SGTs) to users and devices based on their identity and policies. When integrating Cisco ISE with Cisco ASA, SXP (Security Group Tag Exchange Protocol) is used to propagate these SGTs from ISE to the ASA, even if the ASA doesn’t have direct Layer 2 adjacency to the tagged devices.

In this setup, ISE assigns SGTs to endpoints based on authentication and authorization policies. The ASA, acting as an SXP peer, receives these SGT mappings through SXP. With this information, the ASA can enforce fine-grained security policies, such as allowing or denying traffic based on the SGT of the source and destination, enabling context-aware access control and micro-segmentation across the network.

<br>

## Configuring TrustSec

On ISE, first we add the ASA as a Network Access Device (NAD) 

![x](/static/2024-11-17-ise-trustsec-asa/01.png)

<br>

Enable Advanced TrustSec and download the PAC

![x](/static/2024-11-17-ise-trustsec-asa/02.png)

> The TrustSec PAC (Provisioning and Authentication Credential) is a cryptographic credential used for establishing secure communication between ASA and ISE within the Cisco TrustSec architecture. It ensures mutual authentication and encryption between TrustSec components, such as SXP peers.

<br>

Next we add the ASA as an SXP Peers on Work Centers >> SXP >> SXP Devices

![x](/static/2024-11-17-ise-trustsec-asa/03.png)

![x](/static/2024-11-17-ise-trustsec-asa/04.png)

<br>

Then we configure the SGTs to be used, here we will utilize SGT_X and SGT_Y

![x](/static/2024-11-17-ise-trustsec-asa/05.png)

<br>

After that we configure a Policy Sets that will assign an SGT for endpoint connecting to the network

![x](/static/2024-11-17-ise-trustsec-asa/06.png)

<br>

Next on ASA, lets make the CTS Configuration

![x](/static/2024-11-17-ise-trustsec-asa/07.png)

```
aaa-server ISE protocol radius
aaa-server ISE (MANAGEMENT) host 198.18.128.11
 key helena123

cts server-group ISE

cts sxp enable
cts sxp default password helena123
cts sxp connection peer 198.18.128.11 password default mode local listener
```

> CTS configuration on an ASA involves enabling Cisco TrustSec, importing a PAC for authentication with ISE, and configuring SXP to exchange SGT-to-IP mappings for dynamic policy enforcement.

<br>

Run "show cts sxp connections" to see the CTS configuration

![x](/static/2024-11-17-ise-trustsec-asa/08.png)

<br>

On ISE side, we can see on the SXP Devices that ASA is up and running

![x](/static/2024-11-17-ise-trustsec-asa/09.png)

<br>

Next we'll import the PAC to ASA, first copy it to the flash: directory and run the import command

![x](/static/2024-11-17-ise-trustsec-asa/10.png)

```
cts import-pac flash:asa.pac password helena123
```

<br>

Now we can use the SGT to create our firewall policies, here we enable SGT_X to access anything on the Internet while SGT_Y can only access specific host of 4.4.4.2

![x](/static/2024-11-17-ise-trustsec-asa/11.png)

<br>

## Dynamic SGT

After connecting endpoint to the network, ISE now has given it an SGT of "SGT_X"

![x](/static/2024-11-17-ise-trustsec-asa/12.png)

<br>

On the SXP Mappings, we can see now this endpoint is mapped with an SGT

![x](/static/2024-11-17-ise-trustsec-asa/13.png)

<br>

This mapping is propagated to ASA through SXP, allowing ASA to also has the very same IP-to-SGT mappings

![x](/static/2024-11-17-ise-trustsec-asa/14.png)

<br>

And because this endpoint has an SGT_X SGT, it can access the internet without restrictions

![x](/static/2024-11-17-ise-trustsec-asa/15.png)

![x](/static/2024-11-17-ise-trustsec-asa/16.png)

<br>

## Static SGT

Beside getting SGT mapped from Policy Sets, we can also statically assign an SGT to an IP Address on ISE

![x](/static/2024-11-17-ise-trustsec-asa/17.png)

<br>

This SGT mappings are reflected on the ASA as well

![x](/static/2024-11-17-ise-trustsec-asa/18.png)

<br>

Now if we try accessing the internet with endpoint with SGT_Y, we can only access 4.4.4.2

![x](/static/2024-11-17-ise-trustsec-asa/19.png)

<br>








