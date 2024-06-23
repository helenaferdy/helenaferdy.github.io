---
title: Check Point Upgrade
date: 2024-06-22 17:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---


## Upgrade through CLI

Here we have a Security Gateway version R80.40 take 294, we will upgrade this to R81.20 take 634

![x](/static/2024-06-22-checkpoint-upgrade/01.png)

<br>

### Installing Deployment Agent

First set the expert password if has not, and then get into expert mode

![x](/static/2024-06-22-checkpoint-upgrade/02.png)

<br>

Run the command below to allow SFTP access through user admin

```shell
chsh -s /bin/bash admin
```

![x](/static/2024-06-22-checkpoint-upgrade/03.png)

<br>

Then copy the [Deployment Agent](https://support.checkpoint.com/results/download/88806) and the [Upgrade Image](https://support.checkpoint.com/results/download/12440) to the /home/admin/ directory

![x](/static/2024-06-22-checkpoint-upgrade/04.png)

<br>

Next untar the Deployment Agent file

```shell
tar -zxvf DeploymentAgent_000002429_1.tgz 
```

![x](/static/2024-06-22-checkpoint-upgrade/05.png)

<br>

Then run the installer

```shell
rpm -Uhv --force CPda-00-00.i386.rpm 
```

![x](/static/2024-06-22-checkpoint-upgrade/06.png)

> The command "rpm -Uhv --force" is used in Linux systems to forcefully upgrade or reinstall a package regardless of its current version or state. It overrides warnings and dependency checks, potentially causing conflicts if not used carefully.

<br>

Notice that after the DA installation the DAservice is stopped, to restart the service run "$DADIR/bin/dastart"

![x](/static/2024-06-22-checkpoint-upgrade/07.png)

<br>

### Upgrading OS

Exit the expert mode, then locally import the Upgrade Image with this command :

```shell
installer import local /home/admin/Check_Point_R81.20_T634_Fresh_Install_and_Upgrade.tar
```

![x](/static/2024-06-22-checkpoint-upgrade/08.png)

<br>

Next show the imported package

```shell
show installer packages imported 
```

![x](/static/2024-06-22-checkpoint-upgrade/09.png)

<br>

Next verify the package

```shell
installer verify 1
```

![x](/static/2024-06-22-checkpoint-upgrade/10.png)

<br>

Finally run the upgrade

```shell
installer upgrade 1
```

![x](/static/2024-06-22-checkpoint-upgrade/11.png)

<br>

After a reboot, the OS should be successfully upgraded

![x](/static/2024-06-22-checkpoint-upgrade/12.png)

<br>
<br>

## Upgrade through GUI

Here we have a Management Server running R80.40 take 294, we'll upgrade this to R81.20 take 634 as well but through GUI

![x](/static/2024-06-22-checkpoint-upgrade/13.png)

<br>

### Installing Deployment Agent

On Upgrades >> Status and Action, we can see here the Deployment Agent build is 1484

![x](/static/2024-06-22-checkpoint-upgrade/14.png)

<br>

To upgrade the Deployment Agent, click on Install DA then install the package

![x](/static/2024-06-22-checkpoint-upgrade/15.png)

<br>

Now we have Deployment Agent build 2429 installed

![x](/static/2024-06-22-checkpoint-upgrade/16.png)

<br>

### Upgrading OS

To upgrade the OS, hit Import Package then import the Upgrade Image

![x](/static/2024-06-22-checkpoint-upgrade/17.png)

<br>

Next verify the imported image

![x](/static/2024-06-22-checkpoint-upgrade/18.png)

![x](/static/2024-06-22-checkpoint-upgrade/19.png)

<br>

After that, hit upgrade

![x](/static/2024-06-22-checkpoint-upgrade/20.png)

![x](/static/2024-06-22-checkpoint-upgrade/21.png)

<br>

The upgrade process takes a while and the server will reboots in the middle of it

![x](/static/2024-06-22-checkpoint-upgrade/22.png)

![x](/static/2024-06-22-checkpoint-upgrade/23.png)

<br>

After it finishes, we can see the version is now on R81.20 take 634 indicating a successfull upgrade

![x](/static/2024-06-22-checkpoint-upgrade/24.png)

<br>













































