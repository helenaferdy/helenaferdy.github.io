---
title: Cisco ISE Guest Portal
date: 2024-11-05 10:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE, RADIUS]
---

Cisco ISE (Identity Services Engine) Guest Portal is a feature that provides secure network access for guest users through customizable web portals. It helps organizations manage guest authentication, user onboarding, and access policies. 

> * Hotspot Portal <br>
> Quick access for guests without formal registration. <br>
> Often used in public spaces for easy, one-click network access. <br>
> * Registered Portal <br>
> Offers more control and tracking as guests provide details before accessing the network. <br>
> Optionally allows guests to create their own accounts with minimal input. <br>


## Hotspot Guest Portal

To configure a hotspot guest portal, first lets create new Policy Sets where any failure of authentication results in continuing to authorization

![x](/static/2024-11-05-ise-guest-portal/01.png)

<br>

Next create two authz rules, one is for portal redirect and the other is for internet access after the guest has gone through the portal

![x](/static/2024-11-05-ise-guest-portal/02.png)

<br>

The "Hotspot Portal" AuthZ rule points to "HOTSPOT_PORTAL" Profile, which here contains a DACL to only allow ISE access and a Web Redirection to redirect users to the Hotspot Portal

![x](/static/2024-11-05-ise-guest-portal/03.png)

<br>

The redirection task uses an ACL named "ACL_REDIRECT", this has to be present on the NAD, unlike the DACL that will be pushed independently

![x](/static/2024-11-05-ise-guest-portal/03a.png)

<br>

We also configure a "ISE_ONLY" DACL to limit user's access to only ISE Servers and some neccessary servers, like DNS

![x](/static/2024-11-05-ise-guest-portal/04.png)

<br>

Next the other "Inet Access" AuthZ Rule points to "INET_ACCESS" Profile. This profile only contains a DACL to allow internet access

![x](/static/2024-11-05-ise-guest-portal/05.png)

![x](/static/2024-11-05-ise-guest-portal/06.png)

<br>

After that, we configure the Hotspot Portal, here we set a Code of "12345" for the internet access

![x](/static/2024-11-05-ise-guest-portal/07.png)

<br>

Then we also configure for any endpoint that has gone through the portal, add it to the "GuestEndpoints" Endpoint Identity Group

![x](/static/2024-11-05-ise-guest-portal/08.png)

<br>

Now lets try accessing the network

![x](/static/2024-11-05-ise-guest-portal/09.png)

<br>

On the Live Radius Logs, we can see the endpoint is given the "Hotspot Portal" AuthZ rule with DACL "ISE_ONLY"

![x](/static/2024-11-05-ise-guest-portal/10.png)

<br>

On the NAD, the same can be observed, including the redirection link to the portal

![x](/static/2024-11-05-ise-guest-portal/11.png)

<br>

Back on the endpoint, we're now redirected to the Hotspot Portal where we can enter the code for internet access

![x](/static/2024-11-05-ise-guest-portal/12.png)

![x](/static/2024-11-05-ise-guest-portal/13.png)

<br>

And after it was successful, we can now access the network

![x](/static/2024-11-05-ise-guest-portal/13a.png)

<br>

On the Live Logs, we see the CoA (Change of Authorization) is sent to change the AuthZ from "Hotspot Portal" to "Inet Access" aloing with its DACL

![x](/static/2024-11-05-ise-guest-portal/14.png)

<br>

And the endpoint is now added to the "GuestEndpoints" group

![x](/static/2024-11-05-ise-guest-portal/15.png)

<br>

On the NAD, we can see the "INET_ONLY" DACL is now pushed allowing endpoint to access internet

![x](/static/2024-11-05-ise-guest-portal/16.png)

<br>

## Registered Portal

Register Portal is used for guests to access the network using the pre-registered user, to configure that lets create a new Guest Portal named "Helena_Guest_Portal"

![x](/static/2024-11-05-ise-guest-portal/20.png)

<br>


























