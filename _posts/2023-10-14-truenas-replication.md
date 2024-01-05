---
title: TrueNAS - ZFS Replication
date: 2023-10-14 11:30:00 +0700
categories: [Storage, TrueNAS]
tags: [TrueNAS]
---

<br>

ZFS Replication is a feature of the TrueNAS storage management system that allows us to create copies of data stored in ZFS file systems and transfer them to another system. It can synchronize data between systems and perform incremental transfers to optimize data replication.

![x](/static/2023-10-14-truenas-replication/00.png)

<br>
<br>

## Preparing the TrueNAS Systems

Here we have a dataset named dummy_data on node1 that we will replicate to node2

![x](/static/2023-10-14-truenas-replication/01.png)

![x](/static/2023-10-14-truenas-replication/01a.png)

<br>

On Node2, we have a freshly installed TrueNAS Core system, let's create a new pool to contain the replicated data

![x](/static/2023-10-14-truenas-replication/02.png)

<br>

We'll name it helena_backup

![x](/static/2023-10-14-truenas-replication/03.png)

![x](/static/2023-10-14-truenas-replication/03a.png)

<br>

Next on Services, make sure SSH is enabled and login as root is also enabled

![x](/static/2023-10-14-truenas-replication/04.png)

> ZFS Replication uses SSH as the transport layer for replicating dataset

<br>
<br>

## Configuring Replication Task

On node1, go to Replication Task, add new

![x](/static/2023-10-14-truenas-replication/05.png)

<br>

Select the dummy_data dataset inside helena pool for the source location, and the helena_backup pool for the destination

![x](/static/2023-10-14-truenas-replication/06.png)

<br>

Set the schedule, here we set it to run hourly

![x](/static/2023-10-14-truenas-replication/06a.png)

<br>

And that should do it, the Replication Task is created

![x](/static/2023-10-14-truenas-replication/07.png)

<br>

ZFS Replication uses snapshot to do replication, therefore when creating replication task, it'll automatically create a snapshot task that runs with it

![x](/static/2023-10-14-truenas-replication/08.png)

<br>
<br>

## Running Replication Task

Back on the Replication task on node1, select Run Now to execute the replication task immediately

![x](/static/2023-10-14-truenas-replication/09.png)

<br>

After a while, the state will change to finished

![x](/static/2023-10-14-truenas-replication/10.png)

<br>

Move over to the node2, we can see on helena_backup pool there's a new dataset created by the replication task

![x](/static/2023-10-14-truenas-replication/11.png)

<br>

The replication task also creates a snapshot for this dataset

![x](/static/2023-10-14-truenas-replication/12.png)

<br>

To easily see the content of a dataset, we can run a shell command and go to "/mnt/helena_backup/dummy_data_backup"

![x](/static/2023-10-14-truenas-replication/13.png)

<br>

To make this dataset available over SMB, let's enable SMB sharing

![x](/static/2023-10-14-truenas-replication/14.png)

<br>

Now both data are available over SMB

![x](/static/2023-10-14-truenas-replication/15.png)

> by default and for the best, the replicated data on node2 is only available as read-only


<br>
<br>

## Simulating Catastrophic Failure

Now we're gonna simulate a scenario where we lose everything on node1, and we're gonna recover the data from node2.
To do that let's delete the helena pool on node1

![x](/static/2023-10-14-truenas-replication/20.png)

<br>

Now that we no longer has a pool on node1, lets create a new one. This pool will be used to contain the restored data that we pull from node2

![x](/static/2023-10-14-truenas-replication/21.png)

![x](/static/2023-10-14-truenas-replication/21a.png)

<br>

Next create a new replication task with the source of node2's dummy_data_backup dataset and the destination of the newly made helena_recover pool

![x](/static/2023-10-14-truenas-replication/22.png)

<br>

Set it to only run once

![x](/static/2023-10-14-truenas-replication/23.png)

<br>

Save and wait until the task finishes

![x](/static/2023-10-14-truenas-replication/24.png)

<br>

Now we should see the pulled dataset from node2 created by the replication task, along with its snapshot

![x](/static/2023-10-14-truenas-replication/25.png)

![x](/static/2023-10-14-truenas-replication/25a.png)

<br>

Checking the content with shell, we can confirm that the data has been succesfully recovered

![x](/static/2023-10-14-truenas-replication/26.png)

<br>












