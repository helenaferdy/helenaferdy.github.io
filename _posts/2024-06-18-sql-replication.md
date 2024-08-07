---
title: SQL Replication
date: 2024-06-18 07:30:00 +0700
categories: [Windows Server, SQL Server]
tags: [SQL]
---

<br>

SQL replication is the process of copying and distributing data and database objects from one database to another and then synchronizing between databases to maintain consistency. 

**Publication** is a collection of database objects that are to be replicated. **Publisher** is the server that makes the data available for replication. **Subscriber** is the server that receives the replicated data. Other key terms include **Distributor**, which manages the flow of data and ensures synchronization, and **Articles**, which are the individual database objects (like tables or views) within a publication.

<br>

Before setting up SQL Replication, make sure the SQL Server and Agent services are running under the domain service account

![x](/static/2024-06-18-sql-replication/00.png)

<br>

Here's the Employee table on WIN1 that will be replicated to WIN2

![x](/static/2024-06-18-sql-replication/01.png)

<br>

## Distribution

On WIN1, configure the Distribution

![x](/static/2024-06-18-sql-replication/03.png)

<br>

Configure WIN1 as the distributor

![x](/static/2024-06-18-sql-replication/04.png)

<br>

Next configure for the SQL Server Agent Service to start automatically

![x](/static/2024-06-18-sql-replication/05.png)

<br>

Because we don't run a pull method from the subscriber, we can use a local path instead of a network path here for the Snapshot Folder

![x](/static/2024-06-18-sql-replication/06.png)

<br>

Then configure the distribution database

![x](/static/2024-06-18-sql-replication/07.png)

<br>

After that select WIN1 as the publisher

![x](/static/2024-06-18-sql-replication/08.png)

<br>

Then select configure distribution at the end of the wizard

![x](/static/2024-06-18-sql-replication/09.png)

<br>

Hit finish

![x](/static/2024-06-18-sql-replication/10.png)

<br>

## Publication

Still on WIN1, creata a new publication

![x](/static/2024-06-18-sql-replication/12.png)

<br>

Select the database to publish

![x](/static/2024-06-18-sql-replication/13.png)

<br>

Then select the publication type

![x](/static/2024-06-18-sql-replication/14.png)

<br>

Next select the articles to be published, here we select one table and one stored procedure

![x](/static/2024-06-18-sql-replication/15.png)

<br>

For the table filters we'll just skip away

![x](/static/2024-06-18-sql-replication/16.png)

<br>

Select create snapshot immediately

![x](/static/2024-06-18-sql-replication/17.png)

<br>

Then on security, enter the domain service user

![x](/static/2024-06-18-sql-replication/18.png)

<br>

Next select create publication at the end of the wizard

![x](/static/2024-06-18-sql-replication/19.png)

<br>

Give the publication a name, then hit finish

![x](/static/2024-06-18-sql-replication/20.png)

<br>

And now the employee publication has been created

![x](/static/2024-06-18-sql-replication/21.png)

<br>

## Subscription

Now on WIN2, create a new subscription

![x](/static/2024-06-18-sql-replication/22.png)

<br>

Select WIN1 as publisher and select the publication

![x](/static/2024-06-18-sql-replication/23.png)

<br>

For the agent location, we will use push type where the agent runs on the publisher

![x](/static/2024-06-18-sql-replication/24.png)

<br>

Then select the WIN2 as the subscriber and select the subcription database

![x](/static/2024-06-18-sql-replication/25.png)

<br>

Specify the domain service user

![x](/static/2024-06-18-sql-replication/26.png)

<br>

For the sync schedule, select run continuously

![x](/static/2024-06-18-sql-replication/27.png)

<br>

Then select initialize subcription

![x](/static/2024-06-18-sql-replication/28.png)

<br>

Select create subscription at the end of the wizard

![x](/static/2024-06-18-sql-replication/29.png)

<br>

Then hit finish

![x](/static/2024-06-18-sql-replication/30.png)

![x](/static/2024-06-18-sql-replication/31.png)

<br>


## Monitoring Replication

On WIN1, on Replication Monitor we can see we have WIN2 in the subscription running the replication

![x](/static/2024-06-18-sql-replication/32.png)

<br>

We can also see the detailed logs for this particular replication

![x](/static/2024-06-18-sql-replication/33.png)

<br>

Now going back to WIN2, we can see the table has been replicated and available on WIN2

![x](/static/2024-06-18-sql-replication/34.png)

<br>

## Modifying Data

On WIN1, we modify the table data by inserting 10 new rows

![x](/static/2024-06-18-sql-replication/35.png)

<br>

We can see on the Synchronization Status that we have 1 transaction with 10 commands being delivered to WIN2

![x](/static/2024-06-18-sql-replication/36.png)

<br>

And WIN2 now has 20 rows of data

![x](/static/2024-06-18-sql-replication/37.png)

<br>

Now we try deleting 7 rows of data on WIN1

![x](/static/2024-06-18-sql-replication/38.png)

<br>

The Synchronization Status also shows 7 commands being delivered to inform WIN2 about the deletion

![x](/static/2024-06-18-sql-replication/39.png)

<br>

Now WIN2 only has 13 rows of data

![x](/static/2024-06-18-sql-replication/40.png)

<br>

Now lets try stopping the synchronization

![x](/static/2024-06-18-sql-replication/41.png)

<br>

If we modify data while sync is off, the WIN2 will not receive the changes until the sync is back on

![x](/static/2024-06-18-sql-replication/42.png)

<br>

Now lets start the synchronization again

![x](/static/2024-06-18-sql-replication/43.png)

<br>

And we can see the 9 commands change we made is being delivered right after

![x](/static/2024-06-18-sql-replication/44.png)

<br>








