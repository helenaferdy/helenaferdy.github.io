---
title: Checkpoint HA Cluster
date: 2024-08-25 13:00:00 +0700
categories: [Security, Checkpoint]
tags: [Checkpoint]
---

A Check Point HA (High Availability) Cluster is a configuration that combines multiple Check Point firewalls to ensure redundancy and improve reliability. In a Check Point HA Cluster, there are two main modes: **Active/Standby** mode, where one firewall actively handles traffic while the other is on standby to take over if the active unit fails, and **Active/Active** mode, where both firewalls actively handle traffic, sharing the load to enhance performance and availability.

![x](/static/2024-08-26-checkpoint-ha/00.png)

<br>

## Active-Standby (High Availability)

Here we have 2 Checkpoint Firewalls all registered and configured

![x](/static/2024-08-26-checkpoint-ha/01.png)

![x](/static/2024-08-26-checkpoint-ha/02.png)

![x](/static/2024-08-26-checkpoint-ha/03.png)

<br>

Select New >> Cluster, give it cluster name and virtual management address, then select the cluster type which in this case is ClusterXL High Availability

![x](/static/2024-08-26-checkpoint-ha/04.png)

<br>

Add both firewalls as members

![x](/static/2024-08-26-checkpoint-ha/05.png)

<br>

Then configure the Cluster Sync interface

![x](/static/2024-08-26-checkpoint-ha/06.png)

<br>

Next the Virtual IP for the downlink interface

![x](/static/2024-08-26-checkpoint-ha/07.png)

<br>

And the Virtual IP for the uplink interface

![x](/static/2024-08-26-checkpoint-ha/08.png)

<br>

After that, the management interface which we will keep private and stand on their own

![x](/static/2024-08-26-checkpoint-ha/09.png)

<br>

Finally finish the wizard

![x](/static/2024-08-26-checkpoint-ha/10.png)

<br>

The HA has been configured, hit publish and install to management's database

![x](/static/2024-08-26-checkpoint-ha/11.png)

![x](/static/2024-08-26-checkpoint-ha/12.png)

![x](/static/2024-08-26-checkpoint-ha/13.png)

<br>

If for some reason the firewall fails to join cluster, enter the CLI and run "cpconfig" and select "Enable cluster membership"

![x](/static/2024-08-26-checkpoint-ha/16.png)

<br>

Now the HA is fully configured

![x](/static/2024-08-26-checkpoint-ha/18.png)

<br>

Checking the virtual IP is also accessible and the downlink nodes can access the internet

![x](/static/2024-08-26-checkpoint-ha/19.png)

<br>

On CLI, run "cphaprob stat" to see the cluster status

![x](/static/2024-08-26-checkpoint-ha/17.png)

<br>

## Failing Over

Lets simulate a network failure by disabling the uplink and downlink interfaces

![x](/static/2024-08-26-checkpoint-ha/20.png)

<br>

On the logs we can see the traffic is handed over from node 1 to node 2 as the node 2 takes over to become the active firewall

![x](/static/2024-08-26-checkpoint-ha/21.png)

<br>

No drops observed on the client side as the failover takes place

![x](/static/2024-08-26-checkpoint-ha/22.png)

<br>

Running "cphaprob stat" shows that now the node 2 is processing 100% traffic

![x](/static/2024-08-26-checkpoint-ha/23.png)

<br>

Now lets restore the network functionality on node 1, we can see that it has changed status from Down to Standby

![x](/static/2024-08-26-checkpoint-ha/24.png)

<br>

To force a failover back to node 1, run "clusterXL_admin down" on the node 2

![x](/static/2024-08-26-checkpoint-ha/25.png)

<br>

Back on Smartconsole, we can see the node 1 is taking the active role while the node 2 is detected to be down

![x](/static/2024-08-26-checkpoint-ha/26.png)

<br>

And all the traffic is passed back to node 1

![x](/static/2024-08-26-checkpoint-ha/27.png)

<br>

Run "clusterXL_admin up" to make the node 2 go up, and now it is back again in standby mode

![x](/static/2024-08-26-checkpoint-ha/28.png)

<br>
<br>

## Active-Active (Load Sharing)

> * **Multicast Mode** sends all incoming packets to a multicast MAC address that is associated with all Cluster Members. This allows every member in the cluster to receive each packet. The decision function on each Cluster Member determines whether it should process the packet, ensuring that all members can actively participate in handling traffic, optimizing load distribution and network performance.
> * **Unicast Mode** assigns the cluster's Virtual IP address to a single Cluster Member, known as the Pivot. The Pivot receives all incoming packets and then forwards them to other members in the cluster for processing based on a decision function. This mode does not require multicast support in the network infrastructure but may result in less balanced traffic distribution, as the Pivot handles initial packet reception and forwarding duties.

### Multicast Mode

On the HA Properties, change the mode to Load Sharing Multicast

![x](/static/2024-08-26-checkpoint-ha/31.png)

<br>

Now both nodes become active and are evenly load balancing traffic

![x](/static/2024-08-26-checkpoint-ha/32.png)

![x](/static/2024-08-26-checkpoint-ha/33.png)

<br>

### Unicast Mode

On the HA Properties, change the mode to Load Sharing Unicast

![x](/static/2024-08-26-checkpoint-ha/34.png)

<br>

This will cause the pivot (node 1) to only handle 30% traffic while the rest is passed to the node 2

![x](/static/2024-08-26-checkpoint-ha/35.png)

<br>
<br>

## Management HA Cluster

To add another Management Server, deploy a new node and select the role to be Secondary

![x](/static/2024-08-26-checkpoint-ha/36.png)

![x](/static/2024-08-26-checkpoint-ha/37.png)

<br>

On Smartconsole, select Add New Check Point Host, enter the details and select Network Policy Management before establishing SIC (Secure Internal Communication)

![x](/static/2024-08-26-checkpoint-ha/38.png)

<br>

That's it, now we have an Active-Standby Management Server where all configuration is still done from primary while we can see read-only configurations from the secondary

![x](/static/2024-08-26-checkpoint-ha/39.png)

<br>












