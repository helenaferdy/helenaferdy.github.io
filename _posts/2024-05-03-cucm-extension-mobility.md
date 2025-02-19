---
title: CUCM Extension Mobility
date: 2024-05-03 17:30:00 +0700
categories: [Collaboration, Unified Communications Manager (CUCM)]
tags: [Collaboration]
---

CUCM Extension Mobility is a feature that allows users to log in to any Cisco IP phones and temporarily adopt their personal settings, such as phone number, speed dials, and services, regardless of their physical location. Extension Mobility is particularly useful in environments where employees frequently move between different workstations or share phones.

<br>

## Configuring Extension Mobility

On CUCM, go to Device >> Phone Services, and Add New

![x](/static/2024-05-03-cucm-extension-mobility/01.png)

<br>

Give it a name and service URL of "http://CUCM:8080/emapp/EMAppServlet?device=#DEVICENAME#"

![x](/static/2024-05-03-cucm-extension-mobility/02.png)

<br>

Next create the user profile on Device >> Device Settings >> Device Profile

![x](/static/2024-05-03-cucm-extension-mobility/03.png)

<br>

Select the device type, this doesn't have to be precise because the user will be able to login to any device regardless

![x](/static/2024-05-03-cucm-extension-mobility/04.png)

<br>

Create the user profile and give it an extension

![x](/static/2024-05-03-cucm-extension-mobility/06.png)

<br>

Then on the top right corner, select Subscribe/unsubscribe Services

![x](/static/2024-05-03-cucm-extension-mobility/07.png)

<br>

Select subscribe to the Extension Mobility that was created earlier

![x](/static/2024-05-03-cucm-extension-mobility/08.png)

<br>

Do the same on the Phone that will be used for Extension Mobility

![x](/static/2024-05-03-cucm-extension-mobility/10.png)

![x](/static/2024-05-03-cucm-extension-mobility/11.png)

<br>

And check on the Enable Extension Mobility

![x](/static/2024-05-03-cucm-extension-mobility/12.png)

<br>

Lastly, on User Management >> End Users, select the user and bind it to the Device Profile created

![x](/static/2024-05-03-cucm-extension-mobility/09.png)

<br>
<br>

## Testing Extension Mobility

On the Device, login using the user credentials

![x](/static/2024-05-03-cucm-extension-mobility/13.png)

<br>

And we can see that we get the Extension Mobility's extension

![x](/static/2024-05-03-cucm-extension-mobility/14.png)

<br>

On CUCM Device >> Actively Logged In Devices, we can see all the users currently logged in to Extension Mobility

![x](/static/2024-05-03-cucm-extension-mobility/15.png)

<br>





















