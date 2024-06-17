---
title: Cisco ISE Active Directory Integration
date: 2024-05-07 10:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE, Active Directory]
---

<br>

Active Directory (AD) integration allows organizations to leverage their existing AD infrastructure for authentication and authorization purposes within the Cisco ISE environment.

<br>

## Adding Active Directory Server

First lets add an AD server, on Cisco ISE go to Administration >> External Identity Sources

![x](/static/2024-05-07-ise-active-directory/01.png)

<br>

Enter the Domain Credentials and optionally the targeted OU

![x](/static/2024-05-07-ise-active-directory/02.png)

<br>

Here's what the **Helena Users** OU contains, which it has 5 users that we will use for auth

![x](/static/2024-05-07-ise-active-directory/02a.png)

<br>

Back on ISE, make sure the ISE is connected and Operational for its integration to the AD Server

![x](/static/2024-05-07-ise-active-directory/03.png)

<br>

Select Test User to make an auth test with the users within the AD's OU

![x](/static/2024-05-07-ise-active-directory/03a.png)

<br>
<br>

## Importing Domain Groups

Still on the same page, go to Groups and select Add >> Select Groups from Directory, and then select the desired groups

![x](/static/2024-05-07-ise-active-directory/04.png)

![x](/static/2024-05-07-ise-active-directory/05.png)

<br>
<br>

## Configuring Authentication Search List

On Administration >> Identity Source Sequences, create a new profile or just edit the default "All_User_ID_Stores" and add the "AD" identity to it

![x](/static/2024-05-07-ise-active-directory/06.png)

<br>
<br>

## Configuring RADIUS

Now we configure RADIUS to use the AD users to authenticate, on Policy >> Policy Sets, select the existing policy and add the expression to allow AD Users to authenticate

![x](/static/2024-05-07-ise-active-directory/07.png)

<br>

Now on the Network Devices, we should be able to use AD Users to authenticate through RADIUS

![x](/static/2024-05-07-ise-active-directory/08.png)

<br>

On RADIUS Live Logs, we can see all the AD Users that authenticate to the network devices

![x](/static/2024-05-07-ise-active-directory/09.png)

<br>
<br>

## Configuring Admin Login

Now we will also use AD Users as admin logins to the ISE, on Administration >> Admin Access, select the AD Server as the Identity Source

![x](/static/2024-05-07-ise-active-directory/10.png)

<br>

Next on Administrators >> Admin Groups, create a new group pointing to the AD's Domain Users

![x](/static/2024-05-07-ise-active-directory/11.png)

<br>

Then on Authorization >> RBAC Policy, create a new policy that gives AD Users permission to Super Admin Menu and Data Access

![x](/static/2024-05-07-ise-active-directory/12.png)

> RBAC means Role Based Access Control

<br>

Now on the main login page, we should see an option to login from mulitple Identity Sources, which in this case is the existing Internal Users and the newly configured AD Users

![x](/static/2024-05-07-ise-active-directory/13.png)

<br>

























