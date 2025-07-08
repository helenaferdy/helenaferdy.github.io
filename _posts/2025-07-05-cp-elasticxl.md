---
title: Check Point ElasticXL
date: 2025-07-05 07:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

ElasticXL is a new clustering technology in Check Point R82 designed to simplify operations and enhance performance for scalable security gateways. It introduces a Single Management Object (SMO), where one gateway acts as a "pivot" member that automatically synchronizes configuration and software across all other cluster members. 

## Installation

To enable ElasticXL, devices need to be in clean install state, and then we can enable ElasticXL on initial setup on the First node of the cluster.

![x](/static/2025-07-05-cp-elasticxl/01.png)

<br>

On the Gaia Portal, the platform now says 'Check Point ElasticXL'

![x](/static/2025-07-05-cp-elasticxl/02.png)

<br>

Add the device to SmartConsole as a gateway object, not as any cluster object

![x](/static/2025-07-05-cp-elasticxl/03.png)

<br>

And now we have a running ElasticXL cluster with only 1 device

![x](/static/2025-07-05-cp-elasticxl/04.png)

<br>

## Adding Cluster Members 

To add cluster member, all we need to do is install R82 on the security gateway, we don't even need to run the first installation wizard

![x](/static/2025-07-05-cp-elasticxl/05.png)

<br>

The new member should show up as Pending Gateway on the first node

![x](/static/2025-07-05-cp-elasticxl/06.png)

<br>

Add it to existing site

![x](/static/2025-07-05-cp-elasticxl/07.png)

<br>

ElasticXL supports a maximum of 3 Cluster Members on each site

![x](/static/2025-07-05-cp-elasticxl/08.png)

<br>

On SmartConsole side, it doesn't really aware about this cluster, all it knows is there's only one gateway registered, even though actually it is a cluster with 3 members behind it. This is what Check Point calls SMO (Single Management Object)

![x](/static/2025-07-05-cp-elasticxl/09.png)

<br>

There's also a new tool called Insights that can be run on clish, this gives us information regarding the cluster

![x](/static/2025-07-05-cp-elasticxl/10.png)

<br>

If there's a failure on the member 01, the next member will take over the role of management

![x](/static/2025-07-05-cp-elasticxl/11.png)

<br>

The failure event is also shown on the Insights

![x](/static/2025-07-05-cp-elasticxl/12.png)

<br>

If we bring back member 01 up, it'll automatically take over the management role again

![x](/static/2025-07-05-cp-elasticxl/13.png)

<br>

In ElasticXL, one "Pivot" gateway acts as the entry point for all network traffic receiving incoming connections. It then intelligently distributes these connections across all active cluster members using an internal distribution matrix, similar to Maestro. So effectively it's a load sharing active-active configuration. <br>
To see which member handles the traffic, there's a ConnView Tool on the Insights

![x](/static/2025-07-05-cp-elasticxl/14.png)

<br>



























