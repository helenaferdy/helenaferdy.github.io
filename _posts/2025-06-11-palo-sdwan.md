---
title: Palo Alto SD-WAN with Panorama
date: 2025-06-10 20:30:00 +0700
categories: [Security, Palo Alto]
tags: [Palo Alto]
---

Palo Alto SD-WAN enables secure, intelligent branch connectivity by dynamically selecting the best path for traffic across multiple WAN links. Palo Alto SD-WAN must be configured and managed through Panorama, which acts as the central controller for all settings and policies.

![x](/static/2025-06-11-palo-sdwan/00.png)

<br>

## Preparing Environments

First off we install the SD-WAN plugin

![x](/static/2025-06-11-palo-sdwan/02.png)

<br>


Here we have 3 Firewalls managed by our Panorama, one is for Hub and two are for Branches

![x](/static/2025-06-11-palo-sdwan/01.png)

<br>

Next on the Objects, we create 2 shared tags to tag our 2 WAN links

![x](/static/2025-06-11-palo-sdwan/03.png)

<br>

## Basic Network Configuration

### HUB

Then on the Network Template, here we configure SD-WAN Interface Profile based on the 2 tags created

![x](/static/2025-06-11-palo-sdwan/04.png)

<br>

Next on the Network Interface, configure the Primary WAN Interface with IP Address & Next Hop, with SD-WAN enabled

![x](/static/2025-06-11-palo-sdwan/05.png)

<br>

Then select the SD-WAN Interface Profile

![x](/static/2025-06-11-palo-sdwan/06.png)

<br>

Do the same for the Secondary link, and add the Non SD-WAN LAN interface as well

![x](/static/2025-06-11-palo-sdwan/07.png)

<br>

Next create the default inside and outside zones

![x](/static/2025-06-11-palo-sdwan/08.png)

<br>

Then create a Virtual Router configured to pass traffic to both WAN gateways

![x](/static/2025-06-11-palo-sdwan/09.png)

<br>

Enabel BGP, give it Router ID and AS Number

![x](/static/2025-06-11-palo-sdwan/09a.png)

<br>

### Branches

Do the same for the JKT & MKW Branches

![x](/static/2025-06-11-palo-sdwan/10.png)

![x](/static/2025-06-11-palo-sdwan/11.png)

<br>

At this point its a good practice to make sure that all nodes have full mesh connectivity through their WAN links

![x](/static/2025-06-11-palo-sdwan/12.png)

<br>

## Policies

Next we tackle the policies, we will use a shared policy of "any-any" for the sake of simplicity

![x](/static/2025-06-11-palo-sdwan/13.png)

<br>

Then create a Traffic Distribution Profile for link distribution algorithm, enter the 2 link tags created earlier

![x](/static/2025-06-11-palo-sdwan/14.png)

<br>

Then create a SD-WAN Policy, we also use "any-any" to ease up the deployment, and refer the TDP here

![x](/static/2025-06-11-palo-sdwan/15.png)

<br>

## SD-WAN

Now we configure the actual SD-WAN configurations, first off we add the HUB to the SD-WAN Devices

![x](/static/2025-06-11-palo-sdwan/16.png)

<br>

Also add the branc devices with type branch

![x](/static/2025-06-11-palo-sdwan/17.png)

<br>

Next on the VPN Clusters, create an VPN Address Pool, this pool will be used dynamically to form up underlay links between hub and spokes

![x](/static/2025-06-11-palo-sdwan/18.png)

<br>

And finally we can create the VPN Cluster with Hub and Spoke type, add all the devices according to their roles and hit commit & push

![x](/static/2025-06-11-palo-sdwan/19.png)

<br>

## SD-WAN Validation

Here's the topology after the configuration is pushed

![x](/static/2025-06-11-palo-sdwan/00a.png)

<br>

The first thing to see is the plugin automatically creates loopback interface used for overlay routing

![x](/static/2025-06-11-palo-sdwan/20.png)

<br>

Tunnel interfaces are created based on the number of links and spokes in the cluster

![x](/static/2025-06-11-palo-sdwan/21.png)

<br>

SD-WAN Virtual Interfaces are also created mapped to their respective tunnel interfaces and zones

![x](/static/2025-06-11-palo-sdwan/22.png)

![x](/static/2025-06-11-palo-sdwan/23.png)

<br>

On CLI, we can validate the SD-WAN connectivity with "show sdwan connection all"

![x](/static/2025-06-11-palo-sdwan/24.png)

<br>

## Routing Validation

On the vRouter, we can see routing tables to reach the branches with next-hop to the loopback of each firewall advertised through BGP

![x](/static/2025-06-11-palo-sdwan/25.png)

![x](/static/2025-06-11-palo-sdwan/26.png)

<br>

On the client side, connectivity tests are working, thus validating the SD-WAN deployment

![x](/static/2025-06-11-palo-sdwan/27.png)

<br>












