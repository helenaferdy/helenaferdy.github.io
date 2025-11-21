---
title: GCP Classic VPN with Check Point
date: 2025-11-21 07:30:00 +0700
categories: [Cloud, Google Cloud Platform (GCP)]
tags: [GCP, Check Point]
---

In this lab, we build a site-to-site VPN between on-premises Check Point and Google Cloud Platform (GCP) using Classic VPN with static routing. This approach relies on encryption domains instead of tunnel interfaces (VTIs). Encryption domains define which on-prem and GCP subnets are allowed to traverse the tunnel, acting as the traffic selector for Phase 2. Since Classic VPN has no routed interface, this subnet-based matching is the only way to determine what traffic gets encrypted.

<br>

## Configuring Classic VPN on GCP

For this lab, we create a new VPN-VPC with subnet 10.60.0.0/24

![x](/static/2025-11-21-gcp-classic-vpn/01.png)

<br>

Next we create the Classic VPN

![x](/static/2025-11-21-gcp-classic-vpn/02.png)

<br>

Then select the VPC and give it a Public IP Address

![x](/static/2025-11-21-gcp-classic-vpn/03.png)

<br>

After that setup the tunnel, here we set the on-prem's Check Point Public IP Address, the Check Point itself sits behind NAT so it's not a direct public IP

![x](/static/2025-11-21-gcp-classic-vpn/04.png)

<br>

The  we configure the cipher settings, we could specify the encryptions and hash settings but we'll leave the default values for simplicity, and we select the Route based routing and enter the on-prem's local prefix (10.21.0.0/24), then hit create.

![x](/static/2025-11-21-gcp-classic-vpn/05.png)

<br>

The Tunnel should be created

![x](/static/2025-11-21-gcp-classic-vpn/06.png)
![x](/static/2025-11-21-gcp-classic-vpn/07.png)

<br>

This also automatically creates a static route to 10.21.0.0/24 using the VPN as the next hop

![x](/static/2025-11-21-gcp-classic-vpn/08.png)

<br>

## Configuring VPN on Check Point

On CP side, first we enable the VPN Domain of the local subnet

![x](/static/2025-11-21-gcp-classic-vpn/09.png)

<br>

Next we create a new Interoperable Device and enter the GCP's VPN Public IP Address that we got when creating the VPN earlier

![x](/static/2025-11-21-gcp-classic-vpn/10.png)

<br>

On Topology, here we have to enter GCP's prefix because this GCP Classic VPN doesn't use Virtual Tunnel Interface (VTI) but rather an Encryption Domain (Policy Based VPN), where the prefix entered here will decide what destination prefix is sent through the tunnel

![x](/static/2025-11-21-gcp-classic-vpn/11.png)

<br>

After that we create a Star VPN Community, selecting CP as Center Gateway and GCP as the Satellite Gateway

![x](/static/2025-11-21-gcp-classic-vpn/12.png)

<br>

On Encryption we configure the default supported values for GCP Classic VPN

![x](/static/2025-11-21-gcp-classic-vpn/13.png)

<br>

On Tunnel Management, we enable Permanent Tunnels and we select One Tunnel per Subnet

![x](/static/2025-11-21-gcp-classic-vpn/14.png)

<br>

And lastly here we enter the Pre-shared Key

![x](/static/2025-11-21-gcp-classic-vpn/15.png)

<br>

Finally, we create a simple policy rule to allow traffic between the two subnets and push the configurations

![x](/static/2025-11-21-gcp-classic-vpn/16.png)

<br>

## Verifying VPN

On GCP side, we can see the Tunnel is now Established

![x](/static/2025-11-21-gcp-classic-vpn/17.png)

<br>

We can also see the detailed logs for the tunnel negotiation process

![x](/static/2025-11-21-gcp-classic-vpn/18.png)

![x](/static/2025-11-21-gcp-classic-vpn/19.png)

<br>

On CP side, the tunnel is also UP, verifying that this VPN setup is complete

![x](/static/2025-11-21-gcp-classic-vpn/20.png)

<br>

Now we can test by sending some traffic from On-prem to GCP

![x](/static/2025-11-21-gcp-classic-vpn/21.png)

<br>

And also from GCP to On-prem

![x](/static/2025-11-21-gcp-classic-vpn/22.png)

<br>

We can see the traffic going back and forth between these 2 subnets here on Check Point's firewall logs

![x](/static/2025-11-21-gcp-classic-vpn/23.png)

<br>

