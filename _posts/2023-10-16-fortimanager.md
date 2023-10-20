---
title: Fortimanager
date: 2023-10-16 07:30:00 +0700
categories: [Security, Fortigate]
tags: [Fortigate]
---

<br>

## What is Fortimanager?

FortiManager is a platform designed to centralize the management of Fortinet's security products, including firewalls, security appliances, and other network devices. FortiManager provides features for configuration, monitoring, reporting, and automation, making it easier for organizations to manage their network security infrastructure efficiently and securely.

<br>
<br>

## Preparing the installer

On fortinet support, download the latest Fortimanager software

![x](/static/2023-10-16-fortimanager/01.png)

<br>

Then deploy it as usual

![x](/static/2023-10-16-fortimanager/02.png)

<br>

Configure the management interface when deploying the ova

![x](/static/2023-10-16-fortimanager/02a.png)

<br>

ALternatively, we can configure it later on the CLI with these commands

![x](/static/2023-10-16-fortimanager/03.png)

<br>
<br>

## Running Fortimanager

After booting the VM, open the IP Address on web browser <br>
Here use the account used to activate the trial license

![x](/static/2023-10-16-fortimanager/04.png)

<br>

Then after a reboot, we should be able to proceed

![x](/static/2023-10-16-fortimanager/05.png)

<br>

And the Fortimanager is up and running

![x](/static/2023-10-16-fortimanager/06.png)

<br>
<br>

## Adding Devices to Manage

On Device Manager, select Add Device >> Discover Device, enter the IP Address of the target

![x](/static/2023-10-16-fortimanager/07.png)

<br>

Now go to the target device, which in this case is a Fortigate Firewall, select Fabric Connectors, then edit on Central Management

![x](/static/2023-10-16-fortimanager/08.png)

<br>

Here enter the IP Address of the Fortimanager, and hit save

![x](/static/2023-10-16-fortimanager/09.png)

<br>

On Fortimanager Status, select Authorize

![x](/static/2023-10-16-fortimanager/10.png)

<br>

On the following pop up, select Approve

![x](/static/2023-10-16-fortimanager/11.png)

<br>

Now the device is centrally managed with Fortimanager

![x](/static/2023-10-16-fortimanager/12.png)

<br>

And back on Fortimanager, we can see the Fortigate being discovered as a Managed Device

![x](/static/2023-10-16-fortimanager/13.png)

<br>













