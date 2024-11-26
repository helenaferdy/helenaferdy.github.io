---
title: Fortinet SD-WAN
date: 2023-10-16 13:30:00 +0700
categories: [Software Defined Networking, Fortinet SD-WAN]
tags: [SD-WAN, Fortigate]
---

<br>

SD-WAN, or Software-Defined Wide Area Network, is a technology that simplifies and enhances the management and performance of WAN by using software-based control and automation to efficiently route and prioritize network traffic.

<br>
<br>

## Network Topology

In this deployment, we're gonna create a simple SD-WAN configuration to load balance traffic from LAN subnet to the Internet going through 2 ISPs

![x](/static/2023-10-16-forti-sdwan/00.png)

<br>
<br>

## Configuring SD-WAN

First, this is the Interfaces configuration

![x](/static/2023-10-16-forti-sdwan/01.png)

<br>

Next create an SD-WAN Zone

![x](/static/2023-10-16-forti-sdwan/02.png)

<br>

After that craete an SD-WAN Member containing the WAN1 interface and attach it to the Zone

![x](/static/2023-10-16-forti-sdwan/03.png)

<br>

Then create another member for the WAN2

![x](/static/2023-10-16-forti-sdwan/04.png)

<br>

And this is how the SD-WAN configuration ends up being

![x](/static/2023-10-16-forti-sdwan/05.png)

<br>

On the SD-WAN Priority Rules, select the one your heart desires

![x](/static/2023-10-16-forti-sdwan/06.png)

<br>

Next create a Static Route going out to the SD-WAN Interface

![x](/static/2023-10-16-forti-sdwan/07.png)

<br>

And lastly, create a Firewall Policy to allow Local Subnet to reach the Internet through the SD-WAN Interface

![x](/static/2023-10-16-forti-sdwan/07a.png)

<br>
<br>

## Testing SD-WAN

Now from a PC on the local subnet, test the Internet Connectivity

![x](/static/2023-10-16-forti-sdwan/08.png)

<br>

And back on the Fortigate, we can see the traffic flowing through the SD-WAN Interface

![x](/static/2023-10-16-forti-sdwan/09.png)

<br>

We can also see the allowed policy for the internet connectivity which is using the SD-WAN policy created earlier

![x](/static/2023-10-16-forti-sdwan/10.png)

<br>

And because now it's using SD-WAN, the traffic going out to the internet is load-balanced between the two ISPs

![x](/static/2023-10-16-forti-sdwan/11.png)

<br>










