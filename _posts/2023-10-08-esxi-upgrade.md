---
title: ESXi Firmware Upgrade
date: 2023-10-08 11:30:00 +0700
categories: [VMware, ESXi]
tags: [esxi]
---

<br>

## What is ESXi?

ESXi is the software that runs on the bare-metal hypervisor known as VMware ESXi. It is the operating system for managing and running virtual machines on VMware's virtualization platform.

<br>
<br>

## Preparing the upgrade patch

First, download the Upgrade File, here we choose the Offline Zip Bundle

![x](/static/2023-10-08-esxi-upgrade/00.png)

<br>

On the [installation guide](https://docs.vmware.com/en/VMware-vSphere/7.0/rn/vsphere-esxi-70u3d-release-notes.html), we need to take note of the Image Profile Name, as it will be used during installation

![x](/static/2023-10-08-esxi-upgrade/00a.png)

<br>

Then upload the patch to the datastore of the host

![x](/static/2023-10-08-esxi-upgrade/02.png)

<br>
<br>

## Installing the patch

Currently we have an UCS Host with ESXi version 6.5

![x](/static/2023-10-08-esxi-upgrade/01.png)

<br>

Access the Host using CLI, locate where the patch is located within the datastore

![x](/static/2023-10-08-esxi-upgrade/05.png)

<br>

Run this command to check the version

```shell
vmware -v
```

Then this command to switch the Host to Maintenance Mode

```shell
esxcli system maintenanceMode set -e true
```

And this command to run the upgrade

```shell
esxcli software profile update -p ESXi-7.0U3d-19482537-standard  -d /vmfs/volumes/6503ca5d-d9ecc6da-2016-0025b5000006/VMware-ESXi-7.0U3d-19482537-depot.zip  --no-sig-check
```

![x](/static/2023-10-08-esxi-upgrade/06.png)

<br>

The upgrade should take less than a minute, and after succed reboot the Host

![x](/static/2023-10-08-esxi-upgrade/07.png)

<br>

After the reboot, run the command to check the version

![x](/static/2023-10-08-esxi-upgrade/08.png)

<br>

And on ESXi Web UI, we can see the version is now 7.0 Update 3

![x](/static/2023-10-08-esxi-upgrade/09.png)

<br>





















