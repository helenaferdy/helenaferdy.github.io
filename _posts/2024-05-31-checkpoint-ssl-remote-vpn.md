---
title: Check Point SSL Remote Access VPN
date: 2024-05-31 17:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point, VPN]
---

Checkpoint SSL remote access VPN provides secure and encrypted connections for remote users to access corporate networks. It ensures data protection and privacy by using SSL/TLS encryption, allowing users to securely connect to their organization's resources from any location over the internet.

<br>

## Topology

Here's the topology of this lab deployment

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/00.png)

<br>

## Configuring SSL VPN

On Check Point SmartConsole, on Access Control enable Mobile Access

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/01.png)

<br>

Select all the clients and hit next

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/02.png)

<br>

On Portal URL, select the external interface

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/03.png)

<br>

Configure the available applications, here we will only select the Demo App

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/04.png)

<br>

Next we dont do AD integration so just skip away

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/05.png)

<br>

Create a test user for testing purposes

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/06.png)

<br>

And that should do it, hit finish

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/07.png)

<br>
<br>

## Testing SSL Remote Access VPN

Now hit the portal that was set earlier and use the test user

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/08.png)

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/08a.png)

<br>

Then download the Check Point VPN Client and install it

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/09.png)

<br>

Configure a new site pointing to the Check Point's external interface

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/10.png)

<br>

Accept the certificates

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/11.png)

<br>

Here select the authentication method, which we will use Username and Password

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/13.png)

<br>

And now we should be able to connect with the user test

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/15.png)

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/16.png)

<br>
<br>

## Configurng VPN Users

Now that the test user is working, lets configure the actual vpn users. Add new user

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/17.png)

<br>

Then add new group containing the vpn user

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/18.png)

<br>

Next on VPN Communities, select the RemoteAccess VPN and configure the Participant User Groups

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/19.png)

<br>
<br>

## Configuring Policy

Next create a new policy to allow VPN Users to access our Local LAN Network

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/20.png)

<br>
<br>

## Connecting VPN

Now we can connect using the newly made vpn user

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/21.png)

<br>

And we can confirm that the vpn is connected and we can access the local lan network

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/22.png)

<br>

Back on Check Point SmartConsole, we can see the vpn traffic being logged here

![x](/static/2024-05-31-checkpoint-ssl-remote-vpn/23.png)

<br>






















































