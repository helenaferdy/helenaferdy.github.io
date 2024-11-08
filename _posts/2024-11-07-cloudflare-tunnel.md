---
title: Cloudflare Tunnel
date: 2024-11-07 10:30:00 +0700
categories: [Other, Cloudflare Tunnel]
tags: []
---

Cloudflare Tunnel enables secure remote access to internal networks or devices over the internet without requiring open inbound firewall ports. By establishing an outbound connection to Cloudflare's network, the tunnel directs internet traffic through a secure, encrypted pathway to internal systems. This setup allows internal systems to be accessible for remote management while protecting them from direct internet exposure and potential attacks.


<br>

To configure Cloudflare Tunnel, we need a domain name. Here we purchase a $1 domain named "helenalab.site" for this purpose

![x](/static/2024-11-07-cloudflare-tunnel/01.png)

<br>

Next go to Cloudflare, sign up for an account, and register the newly purchased domain

![x](/static/2024-11-07-cloudflare-tunnel/02.png)

<br>

Choose the Free version, this requires a payment method even though we'll not be charged. Here we use PayPal for this

![x](/static/2024-11-07-cloudflare-tunnel/03.png)

<br>

Then hit Continue to Activation

![x](/static/2024-11-07-cloudflare-tunnel/04.png)

<br>

Next we need to point the nameserver to Cloudflare to route internet traffic through Cloudflare’s network before it reaches the internal systems. Copy the nameservers

![x](/static/2024-11-07-cloudflare-tunnel/05.png)

<br>

Then on the domain provider update the DNS configuration by pasting the nameservers. Updating the DNS settings to Cloudflare’s nameservers allows all requests for the domain to go through Cloudflare first, allowing Cloudflare to handle security, traffic management, and tunneling configurations.

![x](/static/2024-11-07-cloudflare-tunnel/06.png)

<br>

After couple minutes the domain should be registered on Cloudflare

![x](/static/2024-11-07-cloudflare-tunnel/07.png)

<br>

Next on Zero Trust >> Network >> Tunnels, add new tunnel

![x](/static/2024-11-07-cloudflare-tunnel/08.png)

<br>

Select Cloudflared

![x](/static/2024-11-07-cloudflare-tunnel/09.png)

> * Cloudflared is used to expose internal services to the internet by creating a secure tunnel from a private network to Cloudflare, making applications accessible externally without opening firewall ports. 
> * WARP Connector is designed for individual devices, routing all of their traffic through Cloudflare to provide secure remote access to internal resources and protect internet browsing.

<br>

Then give it a name

![x](/static/2024-11-07-cloudflare-tunnel/10.png)

<br>

Here we can choose the connector types, the easiest one is Docker so that's we're choosing. Copy the docker command

![x](/static/2024-11-07-cloudflare-tunnel/11.png)

<br>

Run the command on the linux server, here we use RHEL

![x](/static/2024-11-07-cloudflare-tunnel/12.png)

<br>

After that the connector should be listed on Cloudflare Tunnel

![x](/static/2024-11-07-cloudflare-tunnel/13.png)

<br>

Next create a Public Hostname that we will access from the internet, here we're trying to access the vCenter server on the Internal Network over Internet

![x](/static/2024-11-07-cloudflare-tunnel/14.png)

<br>

Just like that now the internal app is accessible on the Internet

![x](/static/2024-11-07-cloudflare-tunnel/15.png)

<br>

We can also have multiple Public Hostnames pointing to different internal servers in the same tunnel

![x](/static/2024-11-07-cloudflare-tunnel/16.png)

![x](/static/2024-11-07-cloudflare-tunnel/17.png)

<br>







