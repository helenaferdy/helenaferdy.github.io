---
title: SQL Linked Server
date: 2024-06-19 07:30:00 +0700
categories: [Windows Server, SQL Server]
tags: [SQL]
---

A Microsoft SQL Server linked server allows one SQL Server to access data from another SQL Server instance or different data sources like Oracle, Access, or Excel. This setup enables querying and integrating data from multiple sources, facilitating data management and consolidation tasks.

<br>

Here on WIN1, we have a dummy database with 1 table named "Conversations", we will try to link this server on WIN2 and access the data in this table.

![x](/static/2024-06-19-sql-linked-server/01.png)

<br>

On WIN2, create a new Linked Server

![x](/static/2024-06-19-sql-linked-server/02.png)

<br>

Point it to WIN1 and select the Server Type

![x](/static/2024-06-19-sql-linked-server/03.png)

<br>

Setup the local login credentials and set it to impersonate, because we have the same credentials on both server

![x](/static/2024-06-19-sql-linked-server/04.png)

> * **Not be made**: Connections will not be allowed for logins not defined in the list above.
> * **Be made without using a security context**: Connections will be made without any security context, meaning no user authentication will be provided.
> * **Be made using the login's current security context**: Connections will use the security context of the currently logged-in user. The remote server will authenticate the user based on the credentials of the local login.
> * **Be made using this security context**: Connections will use a specified remote login and password. This option requires entering the remote login and its password, and the remote server will authenticate based on these credentials.

<br>

Now the WIN1 has been linked to WIN2, and we can query to WIN1 on WIN2

![x](/static/2024-06-19-sql-linked-server/05.png)

![x](/static/2024-06-19-sql-linked-server/06.png)

![x](/static/2024-06-19-sql-linked-server/07.png)

<br>


























