---
title: Alibaba VPN with Check Point
date: 2026-03-10 07:30:00 +0700
categories: [Cloud, Alibaba Cloud]
tags: [Alibaba, Check Point]
---

Alibaba Cloud VPN Gateway is a managed service that enables secure, encrypted connections between our VPC and on-premises environments using the IPsec protocol. In this lab, we established a Route-based VPN tunnel between the helena-vpc on Alibaba Cloud and an on-premises Check Point firewall sitting behind a NAT device. This setup allows the two networks to communicate seamlessly across the public internet.

![x](/static/2026-03-10-alibaba-vpn/00.png)

<br>


## VPC

First we prepare the VPC, we will use the int vSwitch (10.0.2.0/24) inside the helena-vpc as alibaba's protected network. VPN setup requires us to have zone redundancy subnets so we add a dummy vs (10.0.99.0/24) just to satisfy this

![x](/static/2026-03-10-alibaba-vpn/01.png)

<br>

## VPN Gateway

Next we setup the VPN Gateway, create new

![x](/static/2026-03-10-alibaba-vpn/02.png)

<br>

We name it VPN_GW and select the Jakarta Region

![x](/static/2026-03-10-alibaba-vpn/03.png)

<br>

Then we select our VPC and vSwitches

![x](/static/2026-03-10-alibaba-vpn/04.png)

<br>

Here's the configure VPN Gateway, take note of the Public IP Address as we will use it as peer address from the on-prem side

![x](/static/2026-03-10-alibaba-vpn/05.png)

<br>

## Customer Gateway

Next we configure the Customer Gateway, which will be our Check Point firewall on on-prem side

![x](/static/2026-03-10-alibaba-vpn/06.png)

<br>

We create new, give it name and insert the CP's Public IP Address. Note that this device is sitting behind NAT and not directly accessible, so the VPN connection will be initiated from the on-prem side

![x](/static/2026-03-10-alibaba-vpn/07.png)

![x](/static/2026-03-10-alibaba-vpn/08.png)

<br>

## IPsec Connection

After that we configure the IPsec VPN

![x](/static/2026-03-10-alibaba-vpn/09.png)

<br>

We give it name and bind it to our VPN Gateway

![x](/static/2026-03-10-alibaba-vpn/10.png)

<br>

Then we select the Destination Routing Mode and select our Customer Gateway for Primary and Secondary Tunnels

![x](/static/2026-03-10-alibaba-vpn/11.png)

<br>

Here's the IPsec configuration after completing the setup. Notice that we have 2 tunnels by default but we will only use the Primary for simplicity

![x](/static/2026-03-10-alibaba-vpn/12.png)

<br>

## Routing

Now back to VPN Gateway, select Destination-based Route Table and add the CP's internal subnet with our IPsec VPN as the next hop

![x](/static/2026-03-10-alibaba-vpn/13.png)

![x](/static/2026-03-10-alibaba-vpn/14.png)

<br>

On vSwitch's Route Tables, we can see the the route to our VPN is automatically created here

![x](/static/2026-03-10-alibaba-vpn/15.png)

<br>

## Check Point

Now we configure the Check Point side, add an Interoperable Device and point it to Alibaba's Public IP

![x](/static/2026-03-10-alibaba-vpn/16.png)

![x](/static/2026-03-10-alibaba-vpn/17.png)

<br>

Then create a Star VPN Community

![x](/static/2026-03-10-alibaba-vpn/18.png)

![x](/static/2026-03-10-alibaba-vpn/19.png)

![x](/static/2026-03-10-alibaba-vpn/20.png)

![x](/static/2026-03-10-alibaba-vpn/21.png)

<br>

We also need firewall policy to allow traffic between the 2 subnets

![x](/static/2026-03-10-alibaba-vpn/22.png)

<br>

Next we create the Tunnel Interface, use the Interoperable Device as the peer and add unused private subnet for p2p communication

![x](/static/2026-03-10-alibaba-vpn/23.png)

<br>

Then we steer the traffic going to 10.0.2.0/24 to use the VTI as the next hop

![x](/static/2026-03-10-alibaba-vpn/24.png)

<br>

Import the newly made tunnel interface into smartconsole

![x](/static/2026-03-10-alibaba-vpn/25.png)

<br>

And lastly, because our firewall is sitting behind NAT, we need to manually enter the Public IP as our local id so Alibaba can identify it. That should conclude the CP side configuration

![x](/static/2026-03-10-alibaba-vpn/26.png)

<br>

## Verification

On Alibaba IPsec Connections, we now see that our Primary Tunnel is up 

![x](/static/2026-03-10-alibaba-vpn/27.png)

![x](/static/2026-03-10-alibaba-vpn/28.png)

<br>

We can also verify it on Check Point side using Smart View

![x](/static/2026-03-10-alibaba-vpn/29.png)

<br>

On on-prem side, our client on 10.21.0.21 can connect to 10.0.2.93 that sits on Alibaba Cloud using the configured VPN

![x](/static/2026-03-10-alibaba-vpn/30.png)

<br>

So is for the connection for the other way around, verifying that our VPN is able to connect the 2 subnets

![x](/static/2026-03-10-alibaba-vpn/31.png)

<br>

Inside Check Point, the logs for the traffic between these 2 subnets can be observed here

![x](/static/2026-03-10-alibaba-vpn/32.png)

<br>

















