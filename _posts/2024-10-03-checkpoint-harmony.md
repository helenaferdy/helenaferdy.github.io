---
title: Check Point Harmony Endpoint
date: 2024-10-03 09:00:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point Harmony Endpoint (SmartEndpoint) is a security solution designed to protect endpoints such as laptops and mobile devices from advanced threats, including ransomware, malware, and phishing attacks. It offers threat prevention, detection, and response capabilities while integrating with centralized management through SmartEndpoint, allowing security teams to monitor and enforce policies across all endpoints.


## Enabling Harmony Endpoint

On the local Management Server, enable "Endpoint Policy Management"

![x](/static/2024-10-03-checkpoint-harmony/01.png)

<br>

Then install the database for this to take effect

![x](/static/2024-10-03-checkpoint-harmony/02.png)

<br>

## Accessing Endpoint Management Portal

Now the SmartEndpoint Console can be accessed, start with downloading initial client packages

![x](/static/2024-10-03-checkpoint-harmony/03.png)

<br>

Other than console, we can also access the Harmony Web GUI, which is the preferable way to manage the endpoints

![x](/static/2024-10-03-checkpoint-harmony/04.png)

<br>

## Endpoint Client

On Policy >> Endpoint Client, here we can manage the client packages like downloading the latest ones if we have internet connection or uploading a specific packages downloaded from Check Point Support page

![x](/static/2024-10-03-checkpoint-harmony/05.png)

<br>

On Deployment Policy, we can specify the policy to deploy the client version and the activated features

![x](/static/2024-10-03-checkpoint-harmony/06.png)

<br>

To actually install it on the client pc, download the client package

![x](/static/2024-10-03-checkpoint-harmony/07.png)

<br>

Then install it over on the client pc

![x](/static/2024-10-03-checkpoint-harmony/08.png)

![x](/static/2024-10-03-checkpoint-harmony/09.png)

<br>

On the Asset Management, we can see the pc with installed client shows up here

![x](/static/2024-10-03-checkpoint-harmony/10.png)

<br>

## Active Directory Connection

Check Point Harmony integrates with Active Directory to easily manage security by using AD groups. It applies security policies based on users' roles, making deployment and access control simpler and consistent across all devices in the network.

![x](/static/2024-10-03-checkpoint-harmony/13.png)

<br>

Here we can see the AD Groups have been retrieved

![x](/static/2024-10-03-checkpoint-harmony/14.png)

<br>

Now we can use these groups to better taylor our policies

![x](/static/2024-10-03-checkpoint-harmony/15.png)

<br>

## Mixed Operation Mode

In Check Point Harmony, **Computer Mode** and **Mixed Operation Mode** determine how policies are applied to devices: <br>
> * **Computer Mode**: Security policies are applied based on the device itself, regardless of the user. This is useful when multiple users share the same computer.
> * **Mixed Operation Mode**: Policies can be applied to both the device and the individual user. This allows for more flexibility, enabling specific policies for different users on the same device.

![x](/static/2024-10-03-checkpoint-harmony/16.png)

<br>

Now we can make 2 types of policies, for user specific or for the endpoint pc, where if there's conflicts the user policies will take priority.

![x](/static/2024-10-03-checkpoint-harmony/17a.png)

<br>

## Applying Policies

There are many seetings and policies that we can configure here, for example we can enable File Protection to block malware being accessed by all computers

![x](/static/2024-10-03-checkpoint-harmony/18.png)

<br>

After installing the policy, we can see the audit logs for the install operation

![x](/static/2024-10-03-checkpoint-harmony/27.png)

<br>

Now when we try copying potentially malicious files, it'll be blocked

![x](/static/2024-10-03-checkpoint-harmony/19.png)

<br>

We can see this specific block action on the Harmony Logs

![x](/static/2024-10-03-checkpoint-harmony/20.png)

<br>

Another example is to force a URL block rule, here we will block some URLs but for only the user "Ember Spirit"

![x](/static/2024-10-03-checkpoint-harmony/21.png)

<br>

Now if we access those URLs from "Ember Spirit" user, we wont be able to do so, while still letting other users to access it.

![x](/static/2024-10-03-checkpoint-harmony/22.png)

<br>

## Push Operations

We can do multiple push operations to the endpoint PCs, for example here we will kickstart a malware scan

![x](/static/2024-10-03-checkpoint-harmony/11.png)

<br>

After a while to malware scan starts on the client pc

![x](/static/2024-10-03-checkpoint-harmony/12.png)

<br>

Other operation that we can do is unistalling the endpoint client, to do that create a new push operation with uninstall action

![x](/static/2024-10-03-checkpoint-harmony/23.png)

![x](/static/2024-10-03-checkpoint-harmony/24.png)

![x](/static/2024-10-03-checkpoint-harmony/25.png)

<br>

The uninstall action will show up on the targeted machine

![x](/static/2024-10-03-checkpoint-harmony/26.png)

<br>

## High Availabilty

Here we added another management server as the secondary node

![x](/static/2024-10-03-checkpoint-harmony/30.png)

<br>

Accessing the SmartEndpoint shows that now we have 2 Endpoint Servers

![x](/static/2024-10-03-checkpoint-harmony/31.png)

<br>

And we can now also login to the Secondary Server, but with a Read Only permission because all operations are done from the active server

![x](/static/2024-10-03-checkpoint-harmony/32.png)

<br>

For the Harmony Web UI, its not accessible on the secondary server until it becomes active

![x](/static/2024-10-03-checkpoint-harmony/33.png)

<br>

On the Audit Logs we can see the sync traffic between these 2 endpoint servers, meaning the backup server will have the same configuration as the active

![x](/static/2024-10-03-checkpoint-harmony/34.png)

<br>

## Failing Over

Now lets simulate a failure on cpms1 so the active management server switches to cpms2

![x](/static/2024-10-03-checkpoint-harmony/35.png)

<br>

Logging in to the SmartEndpoint on the Seconadry Server, we can see now we have Read/Write permission here

![x](/static/2024-10-03-checkpoint-harmony/37.png)

<br>

And the Harmony Web UI is also now accessible and we can do policy push operations

![x](/static/2024-10-03-checkpoint-harmony/38.png)

<br>

On the client side, it'll also automatically switch to the currently active server

![x](/static/2024-10-03-checkpoint-harmony/36.png)

<br>


