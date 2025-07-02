---
title: Fortigate Destination NAT & Port Forwarding
date: 2023-09-19 09:30:00 +0700
categories: [Security, Fortinet]
tags: [Fortinet]
---

<br>

## What is Destination NAT & Port Forwarding?

NAT (network address translation) is a method that allows devices on a local network to share a single public IP address for accessing resources on the internet.
Destination NAT (DNAT) specifically focuses on modifying the destination IP address of incoming packets to redirect incoming network traffic to a different destination IP address or port number within the local network.

<br>

Port forwarding is a technique used to redirect incoming network traffic from one network port to another, typically on a different device within a local network.

<br>
<br>

## Topology

Here's the topology, where we have a linux host (61.0.0.141) that will be accessed from outside using Destination NAT on Public IP of 198.18.0.141

![x](/static/2023-09-19-forti-dnat/01.png)

<br>
<br>

## Configuring Virtual IP

On Fortigate, go to Policy & Objects >> Virtual IPs, create new

![x](/static/2023-09-19-forti-dnat/02.png)

Set the interface to be the Internet facing (outside), set type to static NAT, and give the respective public and private IP Addresses

![x](/static/2023-09-19-forti-dnat/03.png)

<br>
<br>

## Configuring Firewall Policy

On Policy & Objects >> Firewall Policy, create a policy enabling traffic from Outside to the Virtual IP

![x](/static/2023-09-19-forti-dnat/04.png)

> * Incoming interface will be the Outisde interface.
> * Outgoing interface will be the Inside interface
> * Source will be all
> * Destination will be the Virtual IP
> * Service will be all
> * NAT will be disabled

<br>

Now the Public Virtual IP should be accessible and pointing to the Linux host

![x](/static/2023-09-19-forti-dnat/05.png)

<br>

And the web service on that host is also accessible using the public virtual IP

![x](/static/2023-09-19-forti-dnat/06.png)

<br>
<br>

## Configuring Port Forwarding

On Policy & Objects >> Virtual IPs, edit the Linux_Public, configure the Port Forwarding as below

![x](/static/2023-09-19-forti-dnat/07.png)

<br>

Now we can see this entry has a specific port mapping

![x](/static/2023-09-19-forti-dnat/07a.png)

<br>

And accessing it from port 7000 on the public IP will result going to the web service on port 80 on the linux host

![x](/static/2023-09-19-forti-dnat/08.png)

<br>

On the report, we can see the successful connection coming from Internet to the Virtual IP

![x](/static/2023-09-19-forti-dnat/09.png)

<br>