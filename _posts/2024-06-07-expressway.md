---
title: Cisco Expressway Mobile Remote Access (MRA)
date: 2024-06-07 09:30:00 +0700
categories: [Collaboration, Expressway]
tags: [Expressway, CUCM]
---

Mobile and Remote Access (MRA) allows remote and mobile workers to securely access the corporate collaboration services without the need for a VPN. It facilitates secure connections for voice, video, instant messaging, and presence directly through the Expressway edge and core servers, enabling seamless communication as if users were within the corporate network.

<br>

Here's the topology for this MRA deployment

![x](/static/2024-06-07-expressway/00.png)

<br>

## Installing Expressways

First deploy 2 Expressways VMs, one for Expressway Core and the other for Expressway Edge

![x](/static/2024-06-07-expressway/01.png)

<br>

Run the CLI configuration and the web UI should be accessible right after

![x](/static/2024-06-07-expressway/02.png)

![x](/static/2024-06-07-expressway/03.png)

<br>

### Expressway Core

On the Expressway Core VM, select Expressway-C and for the service select MRA 

![x](/static/2024-06-07-expressway/04.png)

<br>

Configure the IP address, credentials, DNS and NTP

![x](/static/2024-06-07-expressway/05.png)

<br>

And that should do it for the Exp-C configuration

![x](/static/2024-06-07-expressway/06.png)

<br>

### Expressway Edge

For the Edge, select Expressway-E at the beginning and the rest should be roughly the same

![x](/static/2024-06-07-expressway/07.png)

![x](/static/2024-06-07-expressway/08.png)

![x](/static/2024-06-07-expressway/09.png)

<br>

Because Exp-E uses two interfaces, go to System >> Network interfaces and configure the Internal and External interfaces

![x](/static/2024-06-07-expressway/09a.png)

<br>

## Configuring CUCM Integration

> Expressway C

On Core, go to Configuration >> Unified Communications, select the mode to be MRA and configure the authentication method

![x](/static/2024-06-07-expressway/10.png)

<br>

Next add the CUCM servers

![x](/static/2024-06-07-expressway/11.png)

<br>

> Expressway E

On Edge, go to Configuration >> Unified Communications and select the MRA

![x](/static/2024-06-07-expressway/11a.png)

<br>

## Configuring Domains

> Expressway C

On Core, go to Configuration >> Domains, input the domain name the supported services

![x](/static/2024-06-07-expressway/12.png)

<br>

## Dealing with Certificates

> Expressway C

On Core, go to Maintenance >> Security >> Server certificate, select generate CSR

![x](/static/2024-06-07-expressway/13.png)

<br>

Then sign the CSR on the Certificate Authority using Client Server Auth Template

![x](/static/2024-06-07-expressway/14.png)

<br>

Next add the Root CA Certificate on to the Trusted CA Certificates

![x](/static/2024-06-07-expressway/15.png)

<br>

Back to Server certificate, finish the CSR by uploading the signed certificate

![x](/static/2024-06-07-expressway/16.png)

<br>

> Expressway E

Repeat the process for the Edge

![x](/static/2024-06-07-expressway/17.png)

![x](/static/2024-06-07-expressway/18.png)

![x](/static/2024-06-07-expressway/19.png)

![x](/static/2024-06-07-expressway/20.png)

<br>

## Creating Zone

> Expressway E

On the Edge, go to Configuration >> Zones, create the traversal zone pointing to the Core

![x](/static/2024-06-07-expressway/21.png)

<br>

> Expressway C

On the Core, enter the same credential and port as configured on the Edge

![x](/static/2024-06-07-expressway/22.png)

<br>

And add the Edge's address as the peer

![x](/static/2024-06-07-expressway/23.png)

<br>

Now we can see the traversal zone is active

> Expressway C

![x](/static/2024-06-07-expressway/24.png)

> Expressway E

![x](/static/2024-06-07-expressway/25.png)

<br>

## Configuring DNS Srv

On the internet, configure the hostname to point to the Edge External NAT IP Address

![x](/static/2024-06-07-expressway/26.png)

<br>

Then create a DNS Srv _collab-edge that points to the Edge's FQDN

![x](/static/2024-06-07-expressway/27.png)

<br>

Run a nslookup to validate the configuration

![x](/static/2024-06-07-expressway/28.png)

<br>

## Logging in using MRA

Now on the internet, when we try to hit the helena.gg domain, the endpoint will use the _collab-edge DNS Srv Record to locate the CUCM Service

![x](/static/2024-06-07-expressway/29.png)

![x](/static/2024-06-07-expressway/30.png)

<br>

On Maintenance >> Logging, we can see the the logs of the users authenticating to CUCM through Expressway

![x](/static/2024-06-07-expressway/31.png)

<br>









































