---
title: Cisco ISE TrustSec with Palo Alto Panorama
date: 2025-06-09 07:30:00 +0700
categories: [Security, Palo Alto]
tags: [Palo Alto]
---


Palo Alto Networks integrates with Cisco ISE’s pxGrid via Panorama using the Cisco TrustSec plugin to import Security Group Tags (SGTs) and IP-to-SGT mappings. By establishing a secure connection to ISE’s pxGrid controller, Panorama retrieves TrustSec data, enabling Palo Alto firewalls to enforce dynamic, identity-based policies based on Cisco TrustSec classifications. 
> [Reference: Configure the Panorama Plugin for Cisco TrustSec](https://docs.paloaltonetworks.com/panorama/11-1/panorama-admin/panorama-plugins/endpoint-monitoring-for-cisco-trustsec/configure-the-panorama-plugin-for-cisco-trustsec)


## Installing Plugin

Download the Cisco TrustSec Plugin on the support portal

![x](/static/2025-06-09-panorama-ise/01.png)

<br>

On Panorama >> Plugins, upload and install the plugin

![x](/static/2025-06-09-panorama-ise/02.png)

<br>

## Configuring Cisco TrustSec

After installing plugin, a new Cisco TrustSec menu will appear, open the setup and enable monitoring

![x](/static/2025-06-09-panorama-ise/03.png)

<br>

Add new Notify Group, insert the Device Groups we wish to receive this TrustSec configuration

![x](/static/2025-06-09-panorama-ise/04.png)

<br>

## PXGrid Integration

On Cisco ISE, on Administration >> PXGrid >> Settings, make sure to allow password based account creation

![x](/static/2025-06-09-panorama-ise/06.png)

<br>

On Panorama CLI, run this command to create the client account without verifying certificates

```
request plugins cisco_trustsec create-account server-cert-verification-enabled no client-name panorama host 198.18.128.13
```

![x](/static/2025-06-09-panorama-ise/07.png)

<br>

Back on ISE, the panorama client should show up

![x](/static/2025-06-09-panorama-ise/08.png)

<br>

Hit approve so it becomes enabled

![x](/static/2025-06-09-panorama-ise/09.png)

<br>

Next continuing the panorama configuration, add the PxGrid server with the generated credential

![x](/static/2025-06-09-panorama-ise/10.png)

<br>

After that on the Monitoring Definition, add new one refrencing the PxGrid Sever and the Notify Group created earlier

![x](/static/2025-06-09-panorama-ise/11.png)

<br>

Make sure the status is success and synced.

![x](/static/2025-06-09-panorama-ise/12.png)

<br>

> This process is repeated for every PxGrid server

## Using the TrustSec SGT

Now we can create a Dynamic Object and map it to an SGT

```
'cts.svr_<pxgrid server name>.sgt_<sgt name>'

## if more than 1 pxgrid server
'cts.svr_<pxgrid1 server name>.sgt_<sgt name>' or 'cts.svr_<pxgrid2 server name>.sgt_<sgt name>'
```

![x](/static/2025-06-09-panorama-ise/13.png)

<br>

Next we can use the Dynamic Object in the Policy Rules

![x](/static/2025-06-09-panorama-ise/14.png)

<br>

## Validating TrustSec

Here we have an endpoint with SGT Employees connected to the network

![x](/static/2025-06-09-panorama-ise/15.png)

<br>

On the Dynamic Object we created, we will be able to see the SGT-to-IP mappings for the specified SGTs

![x](/static/2025-06-09-panorama-ise/16.png)

<br>









