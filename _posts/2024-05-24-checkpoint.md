---
title: Check Point Security Gateway
date: 2024-05-24 17:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point Security Gateway is a robust security solution offering comprehensive protection for enterprise networks. Combining firewall, intrusion prevention, VPN, and advanced threat prevention capabilities, it safeguards against a wide range of cyber threats.

<br>

## Deployment Topology

There are two types of onpremise installation :
> * Standalone Deployment : This involves a single Node working as the Security Gateway and the Management Server.
> * Distributed Deployment : This distributes the installation into two nodes, Security Gateway and Management Server.

<br>

Here's the topology for this deployment

![x](/static/2024-05-24-checkpoint/00.png)

<br>
<br>

## Installing Check Point Security Gateway

First download the ISO installer on [Check Point Support Center](https://support.checkpoint.com/results/download/124397)

![x](/static/2024-05-24-checkpoint/01a.png)

<br>

Then just deploy it as usual, for the Security Gateway we will use 3 network interfaces

![x](/static/2024-05-24-checkpoint/01.png)

<br>

Run the installer and select Install Gaia

![x](/static/2024-05-24-checkpoint/02.png)

> Gaia is an advanced, secure operating system for its security appliances. It's designed to offer maximum flexibility and efficiency for managing security policies and configurations across various network environments. Gaia integrates the functionalities of different Check Point software blades into a single unified platform, providing a comprehensive security solution for enterprises.

<br>

Then allocate the disk partitions

![x](/static/2024-05-24-checkpoint/03.png)

<br>

Next set the admin password

![x](/static/2024-05-24-checkpoint/04.png)

![x](/static/2024-05-24-checkpoint/05.png)

<br>

Next select the management network interface

![x](/static/2024-05-24-checkpoint/06.png)

<br>

Then set the Management IP Addressing

![x](/static/2024-05-24-checkpoint/07.png)

<br>

Confirm and start the installation

![x](/static/2024-05-24-checkpoint/08.png)

<br>

After a while, the Web UI should be accessible

![x](/static/2024-05-24-checkpoint/09.png)

<br>

Start the configuration wizard

![x](/static/2024-05-24-checkpoint/10.png)

<br>

Hit continue

![x](/static/2024-05-24-checkpoint/11.png)

<br>

Configure the management interface again

![x](/static/2024-05-24-checkpoint/12.png)

<br>

Optionally, configure the uplink interface, but we'll do it later so just hit next

![x](/static/2024-05-24-checkpoint/13.png)

<br>

Configure hostnamem, DNS and NTP stuff

![x](/static/2024-05-24-checkpoint/14.png)

![x](/static/2024-05-24-checkpoint/15.png)

<br>

Next select the installation type

![x](/static/2024-05-24-checkpoint/16.png)

<br>

Then on Products, select only Security Gateway because we're installing the Distributed Deployment

![x](/static/2024-05-24-checkpoint/17.png)

<br>

Then enter the Activation Key that'll later be used to intergrate it with the Management Server

![x](/static/2024-05-24-checkpoint/19.png)

<br>

Review and hit Finish

![x](/static/2024-05-24-checkpoint/20.png)

<br>

After installation finishes, reboot the system

![x](/static/2024-05-24-checkpoint/21.png)

<br>

And now we can log in to the Security Gateway

![x](/static/2024-05-24-checkpoint/22.png)

<br>

Here we configure the WAN, LAN, and Management Interfaces

![x](/static/2024-05-24-checkpoint/23.png)

<br>
<br>

## Installing Check Point Management Server

Using the same ISO Installer, deploy another VM with only one network inteface

![x](/static/2024-05-24-checkpoint/24.png)

<br>

Repeat pretty much everything that was done earlier, with the difference on the Products which we will only select the Security Management

![x](/static/2024-05-24-checkpoint/25.png)

<br>

Review and hit finish

![x](/static/2024-05-24-checkpoint/26.png)

<br>

After a while, the Management Server Web UI should be accessible, and here we have a menu to download the SmartConsole software

![x](/static/2024-05-24-checkpoint/27.png)

<br>
<br>

## Configuring Check Point Smart Console

Download the SmartConsole

![x](/static/2024-05-24-checkpoint/28.png)

<br>

Install and access the Management Server's IP Address through it

![x](/static/2024-05-24-checkpoint/29.png)

<br>

Now the Management Server is up

![x](/static/2024-05-24-checkpoint/30.png)

<br>

To add the Security Gateway, select add Gateway

![x](/static/2024-05-24-checkpoint/31.png)

<br>

Fill in the details and the Activation Key configured earlier

![x](/static/2024-05-24-checkpoint/32.png)

<br>

Now both servers are up and running

![x](/static/2024-05-24-checkpoint/33.png)

<br>
<br>

## Allowing Internet Access

First, create a new network object named "lan network"

![x](/static/2024-05-24-checkpoint/34.png)

<br>

By default Check Point will configure the "hide" NAT Policy, basically meaning it'll mask the newtork with the gateway's IP Address

![x](/static/2024-05-24-checkpoint/35.png)

![x](/static/2024-05-24-checkpoint/36.png)

<br>

Next create a Policy to allow "lan_network" to access internet

![x](/static/2024-05-24-checkpoint/37.png)

<br>

After that, Publish and hit Install Policy

![x](/static/2024-05-24-checkpoint/38.png)

<br>

Now on the user pc, we should have an internet access

![x](/static/2024-05-24-checkpoint/39.png)

<br>

Back on the SmartConsole, on Logs & Monitor we can see the logs for the internet access

![x](/static/2024-05-24-checkpoint/40.png)

<br>

Oh ya, the Smart Console is only available for Windows, so if youre using MacOS then youre out of luck, although we can access the Web Smart Console, it has so many missing menus and features that it just leaves you wondering whats the point? 

![x](/static/2024-05-24-checkpoint/41.png)

<br>










