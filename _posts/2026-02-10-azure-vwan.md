---
title: Azure vWAN
date: 2026-02-10 07:30:00 +0700
categories: [Cloud, Azure]
tags: [Azure]
---

Azure Virtual WAN (vWAN) is a networking service that provides a scale-out connectivity to the Azure global backbone. At the heart of this architecture is the Virtual Hub, a Microsoft-managed central router that acts as the transit point for all connected spokes and security services.

In this lab, we leverage this hub-and-spoke relationship to build a centralized security architecture. By connecting multiple application VNets to the Virtual Hub, we transform our FortiGate VM into an Egress Firewall. This setup ensures that every packet destined for the internet from any spoke is steered through the Fortigate firewall.

![x](/static/2026-02-10-azure-vwan/00.png)

<br>

## Virtual Networks

Following from previous deployment, we already have zelena-vnet configured with 2 subnets for our Fortigate Firewall. Next we will create 2 additional vnets for our apps

![x](/static/2026-02-10-azure-vwan/01.png)

<br>

The first one is app1-vnet on 10.10.0.0/24

![x](/static/2026-02-10-azure-vwan/02.png)

<br>

And the second one is app2-vnet on 10.20.0.0/24

![x](/static/2026-02-10-azure-vwan/03.png)

![x](/static/2026-02-10-azure-vwan/04.png)

<br>

Previously we used Route Tables to steer traffic from int subnet to use fortigate as our default gateway, here we remove that because all routing will be configured on the Virtual Hub

![x](/static/2026-02-10-azure-vwan/04a.png)

<br>

## Virtual WAN

vWAN is a managed networking service that allows us to connect multiple VNets to create a hub-and-spoke architecture. Here we will create a new vWAN

![x](/static/2026-02-10-azure-vwan/05.png)

<br>

Give it name and select the Standard Type

![x](/static/2026-02-10-azure-vwan/06.png)

![x](/static/2026-02-10-azure-vwan/07.png)

<br>

## Virtual Hub

Enter the vWAN and select the Hubs. vHub is the core managed hub inside Virtual WAN that acts as a central routing and connectivity point. Let's create a new one

![x](/static/2026-02-10-azure-vwan/08.png)

<br>

Give it name and provide it with a private subnet that's not used, then select the lowest hub capacity and leave the rest as is

![x](/static/2026-02-10-azure-vwan/09.png)

![x](/static/2026-02-10-azure-vwan/10.png)

![x](/static/2026-02-10-azure-vwan/10a.png)

<br>

Inside the vHub, there's Virtual Network Connections. It is the link that attaches a spoke VNets to the hub and enables the VNet to use the hub for transit routing

![x](/static/2026-02-10-azure-vwan/11.png)

<br>

Lets add all the vnets into this VNC

![x](/static/2026-02-10-azure-vwan/12.png)

> zelena-vnet

![x](/static/2026-02-10-azure-vwan/13.png)

> app1-vnet

![x](/static/2026-02-10-azure-vwan/14.png)

> app2-vnet

<br>

And now we have a vHub that connects to 3 vnets

![x](/static/2026-02-10-azure-vwan/15.png)

<br>

We will use the zelena-vnet as our egress network, for that we will add a static route to steer all traffic to Fortigate's internal ip address. Ensure propagate static ip is checked so this route is propagated to the rest of the spokes

![x](/static/2026-02-10-azure-vwan/16.png)

<br>

We can verify the routing for this hub on Effective Routes, ensuring 10.10.0.0/24 and 10.20.0.0/24 are going to their respective vnets, and 0.0.0.0/0 is going to the zelena-vnet

![x](/static/2026-02-10-azure-vwan/17.png)

<br>

## Testing

Now we can test this setup, here we deploy 2 linux servers for each app-vnet

![x](/static/2026-02-10-azure-vwan/18.png)

<br>

From app1 on 10.10.0.5, we can ping the app2 on 10.20.0.5, proving the inter-spokes connection is working through our hub, and we can also access the internet, verifying that our traffic is steered to the zelena-vnet to use our Fortigate as egress firewall

![x](/static/2026-02-10-azure-vwan/19.png)

<br>

On the Fortigate side, we can see the traffic coming in from app1 and app2 going out to the internet

![x](/static/2026-02-10-azure-vwan/20.png)

<br>


