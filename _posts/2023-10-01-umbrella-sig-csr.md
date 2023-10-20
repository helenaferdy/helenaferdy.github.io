---
title: Cisco Umbrella - Secure Internet Gateway (SIG) with CSR/ISR Router
date: 2023-10-01 07:30:00 +0700
categories: [Security, Cisco Umbrella]
tags: [Umbrella, SIG, Proxy]
---

<br>

## What is Secure Internet Gateway?

A Secure Internet Gateway (SIG) is a cloud-based service that provides secure and filtered access to the internet for users and devices within an organization. <br>
SIG uses tunnels to establish secure connections between an organization's network and the cloud-based security service. This tunnel encrypts the traffic between the organization's network and the SIG service, ensuring the confidentiality and integrity of data as it travels over the internet.

![x](/static/2023-10-01-umbrella-sig-csr/00.png)

<br>
<br>

## Deployment Topology

On this deployment, we will establish a IPsec Tunnel connection from the CSR1000V Router to the Umbrella Cloud, making all the internet traffic coming form the Inside Segment will be routed to the Umbrella first before heading to the Internet

![x](/static/2023-10-01-umbrella-sig-csr/00a.png)

<br>
<br>

## Configuring Network Tunnel on Umbrella

On Cisco Umbrella, go to Deployments >> Network Tunnels, add new for our CSR router

![x](/static/2023-10-01-umbrella-sig-csr/00b.png)

<br>

Here's after the tunnel added, the entry is present with status of Unestablished

![x](/static/2023-10-01-umbrella-sig-csr/01.png)

<br>
<br>

## Configuring Network Tunnel on CSR Router

Cisco actually provides a great deployment guide [here](https://docs.umbrella.com/umbrella-user-guide/docs/add-a-tunnel-cisco-isr) to configure tunnel on CSR/ISR Routers

![x](/static/2023-10-01-umbrella-sig-csr/01a.png)

<br>

First we define a keyring named "umbrella-kr" for storing pre-shared keys used in the IKEv2 negotiation

```shell
crypto ikev2 keyring umbrella-kr
  peer umbrella
  address 146.112.113.8
  pre-shared-key <password set on umbrella>
```

> * Set the address to the closest umbrella DC, check out [this document](https://docs.umbrella.com/umbrella-user-guide/docs/cisco-umbrella-data-centers) for the details
> * In IKEv2 (Internet Key Exchange version 2), a keyring is used to store pre-shared keys (PSKs) and associate them with specific remote peers for authentication during the VPN negotiation process.

<br>

Next we define an IKEv2 profile named "umbrella-profile" for configuring the IKEv2 VPN connection

```shell
crypto ikev2 profile umbrella-profile
  match identity remote address 146.112.0.0 255.255.0.0
  identity local email <email id set on umbrella>
  authentication remote pre-share
  authentication local pre-share
  keyring local umbrella-kr
  dpd 10 3 periodic
```

<br>

Next define an IPSec transform set named "umbrella-ts" and set the mode to be "tunnel"

```shell
crypto ipsec transform-set umbrella-ts esp-gcm 256 
 mode tunnel
```

> * IPSec transform set specifies the encryption and authentication algorithms and other parameters used for securing IP traffic within an IPSec VPN tunnels

<br>

After that define an IPSec profile named "umbrella" to use the "umbrella-ts" transform set for encryption and the "umbrella-profile" IKEv2 profile for the VPN negotiation

```shell
crypto ipsec profile umbrella
 set transform-set umbrella-ts
 set ikev2-profile umbrella-profile
```

<br>

Next define an IKEv2 proposal named "umbrella-proposal" to specify parameters for the IKEv2 negotiation

```shell
crypto ikev2 proposal umbrella-proposal
 encryption aes-cbc-256
 integrity sha256
 group 14
```

<br>

Next define an IKEv2 policy named "umbrella" to configure IKEv2 parameters matching the outside interface IP Address and to use the "umbrella-proposal" created earlier

```shell
crypto ikev2 policy umbrella
 match address local 198.18.0.22
 proposal umbrella-proposal
```

<br>

Then define a tunnel interface named "Tunnel1" with the source of the outside interface and remote destination of chosen Umbrella DC

```shell
interface Tunnel1
 ip unnumbered GigabitEthernet1
 ip tcp adjust-mss 1280
 tunnel source GigabitEthernet1
 tunnel mode ipsec ipv4
 tunnel destination 146.112.113.8
 tunnel protection ipsec profile umbrella
```

<br>

Next create an access-list for the tunnel access

```shell
ip access-list extended traffic-to-umbrella
  permit ip any any
```

<br>

Then define a route map named "umbrella-route-map" to match the access-list "traffic-to-umbrella" for policy-based routing

```shell
route-map umbrella-route-map permit 10
  match ip address traffic-to-umbrella
  set interface Tunnel1
```

<br>

Lastly,  apply the route-map to Inside Interface 

```shell
interface GigabitEthernet 2
 ip policy route-map umbrella-route-map
```

<br>

And thats pretty much it

<br>
<br>

## Verfying Tunnel Status

On CSR, run "show crypto session detail" to see the tunnel connection

![x](/static/2023-10-01-umbrella-sig-csr/03.png)

<br>

And on Umbrella, we should see the connection also becomes active

![x](/static/2023-10-01-umbrella-sig-csr/02.png)

<br>

Now all the internet traffic from the inside segment will be routed to the Umbrella as the Secure Internet Gateway

![x](/static/2023-10-01-umbrella-sig-csr/04.png)

Here we can see the reports of all traffic coming in from the inside segment of 10.99.0.0/24 being inspected by Cisco Umbrella before being forwarded to the actual destination

<br>





