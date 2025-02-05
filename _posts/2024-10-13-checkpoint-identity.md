---
title: Check Point Identity Awareness
date: 2024-10-13 09:00:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point Identity Awareness allows network administrators to associate users with network activity by integrating user identities with security policies. It enables granular access control by identifying users and their devices, improving visibility and security management.
> * Web Captive Portal : A web-based authentication interface that prompts users for their credentials when accessing the network, commonly used in guest or BYOD environments.
> * Identity Agent : A lightweight client installed on users' devices to provide seamless and continuous user authentication to the network.
> * Identity Collector: Gathers user identity information from various sources, such as Active Directory or Radius servers, and integrates it into the Check Point firewall for policy enforcement.

# Enabling Identity Awareness

On the Security Gateway, enable Identity Awareness and select both AD Query and Browser-based Authentication

![x](/static/2024-10-13-checkpoint-identity/01.png)

<br>

Then enter the Domain along with the Domain Admin credentials

![x](/static/2024-10-13-checkpoint-identity/02.png)

<br>

And enter the Captive Portal URL, which is usually the internal network link

![x](/static/2024-10-13-checkpoint-identity/03.png)

<br>

Then hit finish and publish the changes

![x](/static/2024-10-13-checkpoint-identity/04.png)

<br>
<br>

# Web Captive Portal

> The Web Captive Portal works by redirecting users to a login page when they attempt to access the network. Before granting network access, the portal prompts users to enter their credentials (such as a username and password). Once authenticated, the user's identity is linked to their network activity, allowing access based on predefined security policies.

First lets create a policy containing a Access Role pointing to the "Helena Users" OU in helena.gg domain

![x](/static/2024-10-13-checkpoint-identity/05.png)

<br>

Taking a look into the OU, these are the users allowed to access the internet

![x](/static/2024-10-13-checkpoint-identity/06.png)

<br>

Now if we try accessing internet on the client pc, we'll get redirected to a login portal

![x](/static/2024-10-13-checkpoint-identity/07.png)

![x](/static/2024-10-13-checkpoint-identity/08.png)

<br>

After logging in, this specific IP Address will be associated with a Domain User and thus granted internet access

![x](/static/2024-10-13-checkpoint-identity/09.png)

<br>

Looking at the logs, this ip is now detected to be used by user "Ember Spirit"

![x](/static/2024-10-13-checkpoint-identity/10.png)

<br>
<br>

# Identity Agent

> The Identity Agent is a lightweight client installed on a user's device that continuously authenticates the user to the Check Point firewall. It provides seamless user identification without requiring repeated logins. 

First lets install the Identity Agent endpoint software and point it to the Security Gateway

![x](/static/2024-10-13-checkpoint-identity/11.png)

<br>

After that now we can enter the Domain Credentials that'll be used for internet access

![x](/static/2024-10-13-checkpoint-identity/12.png)

<br>

After that we just automatically are granted an internet access, because our user is part of the OU Domain

![x](/static/2024-10-13-checkpoint-identity/13.png)

<br>

Looking at the logs, its shown that the machine's IP Address is now associated with the user "Invoker"

![x](/static/2024-10-13-checkpoint-identity/14.png)

<br>
<br>

# Identity Collector

> The Identity Collector is a tool that gathers user identity information from various external sources, such as Active Directory, LDAP, or RADIUS servers. It collects and consolidates this data, then sends it to the Check Point firewall for real-time user identification. This allows user authentication done wihout the user entering credentials on login portal or installing any agent software on their machines.

First we need to enable the Identity Collector on the Security Gateway, add the Trusted Client, and give it a secret key

![x](/static/2024-10-13-checkpoint-identity/15.png)

<br>

Next on the trusted client, which in this case is the Domain Controller, we install the Identity Collector

![x](/static/2024-10-13-checkpoint-identity/16.png)

<br>

Then create a Identity Source, which is our helena.gg AD Domain

![x](/static/2024-10-13-checkpoint-identity/17.png)

<br>

Next create a query to fetch all the sources within the domain

![x](/static/2024-10-13-checkpoint-identity/18.png)

<br>

Lastly create a Gateway that uses the query just made and point it to the Security Gateway

![x](/static/2024-10-13-checkpoint-identity/19.png)

<br>

Now on any client pc with eligible domain user, they'll be granted internet access without doing any additional steps

![x](/static/2024-10-13-checkpoint-identity/20.png)

<br>

As it is reflected on the logs showing that the Security Gateway is now aware about the machine's domain user thus granting it internet access

![x](/static/2024-10-13-checkpoint-identity/21.png)

<br>




































