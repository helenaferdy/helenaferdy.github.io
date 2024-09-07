---
title: Check Point VSLS (Virtual System Load Sharing)
date: 2024-09-06 09:00:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point VSLS (Virtual System Load Sharing) is a feature within the VSX environment that allows load balancing of virtual systems (VSs) across multiple physical security gateways in a clustered configuration. In a VSLS cluster, each VS can be assigned to run on a specific gateway, allowing the distribution of traffic load across all available devices, rather than having all VSs active on a single gateway.

![x](/static/2024-09-06-checkpoint-vsls/00.png)

<br>

## Installing MDS

> MDS (Multi-Domain Security Management) is a centralized management solution designed for large-scale environments that require the management of multiple independent security domains. MDS allows administrators to manage multiple Check Point Security Management Servers (referred to as Domains or CMAs—Customer Management Add-ons) from a single, unified platform.

To deploy an MDS, select Multi-Domain Server on initial config

![x](/static/2024-09-06-checkpoint-vsls/01.png)

![x](/static/2024-09-06-checkpoint-vsls/02.png)

![x](/static/2024-09-06-checkpoint-vsls/03.png)

![x](/static/2024-09-06-checkpoint-vsls/04.png)

<br>

Access it on SmartConsole and we will see the default Global Domain. Lets create a new Main Domain

![x](/static/2024-09-06-checkpoint-vsls/05.png)

<br>

Then connect to the domain

![x](/static/2024-09-06-checkpoint-vsls/06.png)

<br>

## Adding VSX Cluster

Before adding the CP into the cluster, first enable "Per Virtual System State" on both nodes

![x](/static/2024-09-06-checkpoint-vsls/07.png)

<br>

Next select New >> VSX >> Cluster, give it name and Virtual Management IP Address

![x](/static/2024-09-06-checkpoint-vsls/08.png)

<br>

Then add the members

![x](/static/2024-09-06-checkpoint-vsls/09.png)

<br>

Next select the Sync Interface and give it a Private Non-routable IP Addresses

![x](/static/2024-09-06-checkpoint-vsls/11.png)

<br>

Then select the desired policy rules to be enabled and hit finish

![x](/static/2024-09-06-checkpoint-vsls/12.png)

![x](/static/2024-09-06-checkpoint-vsls/13.png)

<br>

And now we have the VSX Cluster up and running with 2 members

![x](/static/2024-09-06-checkpoint-vsls/14.png)

<br>

## Creating Site A Domain

Now we have the Cluster running on the Main Domain, lets create a new Domain for Site A

![x](/static/2024-09-06-checkpoint-vsls/15.png)

<br>

Here we create a new VS using the VSX Cluster created on Main Domain

![x](/static/2024-09-06-checkpoint-vsls/16.png)

<br>

Give it IP Addresses and Default Gateway

![x](/static/2024-09-06-checkpoint-vsls/17.png)

<br>

And after a while the VS will be up and running

![x](/static/2024-09-06-checkpoint-vsls/18.png)

<br>

## Creating Site B Domain

Pretty much repeat the process for the Site B Domain

![x](/static/2024-09-06-checkpoint-vsls/19.png)

![x](/static/2024-09-06-checkpoint-vsls/20.png)

<br>

And on the MDS Gateways & Servers, we can see all the nodes we have configured

![x](/static/2024-09-06-checkpoint-vsls/21.png)

<br>

## Load Sharing

At this time, if we run "cphaprob stat" on either VSX Member, we can see that only one node is active for both VS

![x](/static/2024-09-06-checkpoint-vsls/22.png)

<br>

To enable VS Load Sharing, go to MDS Expert Mode and change context to the Main Domain with command "mdsenv"

![x](/static/2024-09-06-checkpoint-vsls/23.png)

<br>

Then run "vsx_util convert_cluster" and select "Distribute All Virtual Systems"

![x](/static/2024-09-06-checkpoint-vsls/24.png)

<br>

And after that, now the VSes are running on each members with 50% load distribution

![x](/static/2024-09-06-checkpoint-vsls/25.png)

<br>

Additionally, we can run "vsx_utl vsls" to change the load sharing configuration, or just to see the current load sharing configuration

![x](/static/2024-09-06-checkpoint-vsls/26.png)

<br>

If we try shutting down the Node 1, we can see all 100% load is now handled by Node 2

![x](/static/2024-09-06-checkpoint-vsls/27.png)

<br>











