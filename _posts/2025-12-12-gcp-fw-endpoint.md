---
title: GCP Firewall Endpoint for IPS & URL Filtering
date: 2025-12-12 07:30:00 +0700
categories: [Cloud, Google Cloud Platform (GCP)]
tags: [GCP]
---

A **Firewall Endpoint** is a managed, zonal resource within Cloud NGFW Enterprise that performs deep packet inspection (Layer 7) using Palo Alto Networks threat prevention technology. By configuring hierarchical firewall policies to "intercept" and redirect specific traffic flows to this endpoint, we enable inline Intrusion Prevention (IPS) and URL Filtering to block advanced threats and enforce compliance directly within the VPC fabric.

<br>

## Security Profiles

First we need to create Security Profiles, first one is for IPS Profile. Here we set the Deny Override for pretty much every alert apart for Informational

![x](/static/2025-12-12-gcp-fw-endpoint/01.png)

<br>

Next we create a URL Filtering Profile, here we'll whitelist 2 URLs while blocking the rest

![x](/static/2025-12-12-gcp-fw-endpoint/02.png)

<br>

Now we have both our Security Profiles configured

![x](/static/2025-12-12-gcp-fw-endpoint/03.png)

<br>

After that we'd need to create a Security Profile Group to contain those 2 profiles

![x](/static/2025-12-12-gcp-fw-endpoint/04.png)

![x](/static/2025-12-12-gcp-fw-endpoint/05.png)

<br>

## Firewall Endpoint

Next we can setup the Firewall Endpoint, here we select the region (FW Endpoint is regional), give it name and select the Billing Project

![x](/static/2025-12-12-gcp-fw-endpoint/06.png)

<br>

For Jumbo frames we'll leave the default

![x](/static/2025-12-12-gcp-fw-endpoint/07.png)

<br>

And for association, we'll associate this FW Endpoint to the ext-vpc, the internet facing VPC, and then hit create

![x](/static/2025-12-12-gcp-fw-endpoint/08.png)

<br>

After about 20 minutes, the Firewall Endpoint is now up and running

![x](/static/2025-12-12-gcp-fw-endpoint/09.png)

<br>

## Firewall Policy

Next we'll create a Firewall Policy

![x](/static/2025-12-12-gcp-fw-endpoint/10.png)

<br>

And then create the rules where we attach the Security Profile Groups

![x](/static/2025-12-12-gcp-fw-endpoint/11.png)

<br>

For mirroring we'll leave it as is

![x](/static/2025-12-12-gcp-fw-endpoint/12.png)

<br>

And for network association we'll select our ext-vpc, and then hit create

![x](/static/2025-12-12-gcp-fw-endpoint/13.png)

<br>

Here's the configure Firewall Policy

![x](/static/2025-12-12-gcp-fw-endpoint/14.png)

<br>

With 2 rules for egress & ingress applying Security Profile Group for IPS & URL Filtering

![x](/static/2025-12-12-gcp-fw-endpoint/15.png)

<br>


## Testing

First from a VM inside GCP, we are now only allowed to access the whitelisted URLs while the rest will result in a failure

![x](/static/2025-12-12-gcp-fw-endpoint/16.png)

<br>

We can see the logs for this event inside the firewall policy

![x](/static/2025-12-12-gcp-fw-endpoint/17.png)

<br>

And for testing the IPS, we will try to attach the VM inside GCP from outside network, here we use a Kali Linux to send a simple Denial of Service (DOS) attack by attempting to exhaust the VM's connection pool by keeping sockets open

![x](/static/2025-12-12-gcp-fw-endpoint/18.png)

<br>

On the GCP Threat Events, we can see that the Firewall Endpoint has successfully identified and prevented this attack

![x](/static/2025-12-12-gcp-fw-endpoint/19.png)

<br>





