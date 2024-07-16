---
title: CUCM Calling in Webex App 
date: 2024-07-16 09:30:00 +0700
categories: [Collaboration, Unified Communications Manager (CUCM)]
tags: [CUCM]
---


Cisco Unified Communications Manager (CUCM) Webex App Calling integrates CUCM with Webex, allowing users to make and receive calls through the Webex App while registered to On Premise CUCM and managed via the Webex Control Hub.

![x](/static/2024-07-16-cucm-webex-hub/00.png)

<br>

## Setting Up Users

On Webex Control Hub, go to Users >> Manage Users >> Manually Add Users

![x](/static/2024-07-16-cucm-webex-hub/01.png)

<br>

Then fill in names and email and keep the rest as default

![x](/static/2024-07-16-cucm-webex-hub/02.png)

![x](/static/2024-07-16-cucm-webex-hub/03.png)

![x](/static/2024-07-16-cucm-webex-hub/04.png)

<br>

Now we have the user added on Control Hub with Not Verified status

![x](/static/2024-07-16-cucm-webex-hub/05.png)

<br>

An invitation email will be sent to activate the user's Webex Account

![x](/static/2024-07-16-cucm-webex-hub/06.png)

![x](/static/2024-07-16-cucm-webex-hub/07.png)

<br>

Now the user is shown as active on Control Hub

![x](/static/2024-07-16-cucm-webex-hub/08.png)

<br>

And that same user can now login to Webex App

![x](/static/2024-07-16-cucm-webex-hub/09.png)

<br>


## Enabling CUCM Calling

On that user >> Licenses, edit the Calling License and select Register to Unified Communication Manager (UCM)

![x](/static/2024-07-16-cucm-webex-hub/10.png)

![x](/static/2024-07-16-cucm-webex-hub/11.png)

<br>

Now on the Webex App, the user will have an option to register to phone service

![x](/static/2024-07-16-cucm-webex-hub/12.png)

<br>

Fill in the CUCM details, whether its a Direct On-Premise Connection or through Expressway's Mobile Remote Access (MRA)

![x](/static/2024-07-16-cucm-webex-hub/13.png)

<br>

Then login with the CUCM user, this doesn't have to have any correlations whatsoever with the Webex User

![x](/static/2024-07-16-cucm-webex-hub/14.png)

![x](/static/2024-07-16-cucm-webex-hub/15.png)

<br>

Now we can make CUCM calls in the Webex App

![x](/static/2024-07-16-cucm-webex-hub/16.png)

<br>

## AD Synchronization with Directory Connector

Instead of manually adding user, we can sync users from Active Directory with Directory Connector. First on Organization Settins >> Cisco Directory Connector, select Download Directory Synchronization App

![x](/static/2024-07-16-cucm-webex-hub/17.png)

<br>

Then deploy it on any regular Windows Server Machine within the Organization

![x](/static/2024-07-16-cucm-webex-hub/18.png)

<br>

After that make sure the service is running under a Domain User

![x](/static/2024-07-16-cucm-webex-hub/19.png)

<br>

Open the Directory Connector and enter the domain name

![x](/static/2024-07-16-cucm-webex-hub/20.png)

<br>

On Configuration >> General, select the Domain Controller

![x](/static/2024-07-16-cucm-webex-hub/21.png)

<br>

Then on Object Selection, select User because we want to sync users and put an LDAP filter to narrow the search scope to a specific Domain Security Group named "Webex Users"

```shell
(&(sAMAccountName=*)(memberOf=CN=Webex Users,OU=Helena Users,DC=helena,DC=gg))
```

![x](/static/2024-07-16-cucm-webex-hub/22.png)

<br>

On User Attribute Mapping, the only mandatory attribute here to fill is the "UID" and here we choose to fill it with the "mail" attribute

![x](/static/2024-07-16-cucm-webex-hub/23.png)

<br>

That should do it for the configuration, now perform the Dry Run

![x](/static/2024-07-16-cucm-webex-hub/24.png)

<br>

On the Dry Run Report we can see the 2 Object Users to be added that match the LDAP search

![x](/static/2024-07-16-cucm-webex-hub/25.png)

<br>

This matches the "Webex Users" Group on the AD that contains those same users

![x](/static/2024-07-16-cucm-webex-hub/26.png)

<br>

Lastly to actually sync the users to Webex Control Hub, select Start Incremental Sync

![x](/static/2024-07-16-cucm-webex-hub/27.png)

<br>

And now the Users are listed on Control Hub with status of Not Verified, select Send Activation Emails to send email activation to these users

![x](/static/2024-07-16-cucm-webex-hub/28.png)

<br>

The user will receive an email to activate the Webex Account

![x](/static/2024-07-16-cucm-webex-hub/29.png)

<br>

Now these users are activated

![x](/static/2024-07-16-cucm-webex-hub/30.png)

<br>

And they can also login to Webex App and use the CUCM Calling phone service

![x](/static/2024-07-16-cucm-webex-hub/31.png)

<br>











