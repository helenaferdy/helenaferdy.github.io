---
title: Cisco ISE TrustSec with FTD
date: 2024-11-17 10:30:00 +0700
categories: [Security, Cisco Firepower Threat Defense (FTD)]
tags: [FMC, FTD, ISE]
---

Cisco ISE integrates with FTD through FMC to leverage Security Group Tags (SGTs) for policy enforcement. ISE assigns SGTs to users and devices based on authentication, while FMC configures FTD to apply security policies based on these SGTs, enabling dynamic, identity-based access control for network traffic and enhancing visibility and security.


## SGT Configurations

Firstly on ISE, lets create the SGT that will be used. Here the SGTs are SGT_X and SGT_Y

![x](/static/2024-11-17-ise-trustsec-asa/05.png)

<br>

Next on Policy Sets, we set here so ISE gives dynamic SGT assignments

![x](/static/2024-11-17-ise-trustsec-asa/06.png)

<br>


## Certificates

ISE-FMC integration requires certificate-based authentication to provide mutual trust between two devices. Here make sure PxGrid has a CA Signed Certificate with Client/Server Authentication enabled on the Certificate Template

![x](/static/2024-11-17-ise-trustsec-fmc/01.png)

<br>

For FMC, we're also gonna give the same type certificate, first lets create a CSR using Certtools and download the CSR & Private Key

![x](/static/2024-11-17-ise-trustsec-fmc/02.png)

<br>

Then sign it on CA Server with Client/Server Authentication Template

![x](/static/2024-11-17-ise-trustsec-fmc/03.png)

<br>

On FMC >> Objects >> PKI >> Trusted CAs, import the Root-CA so we can trus the PxGrid certificate which was issued by thie CA

![x](/static/2024-11-17-ise-trustsec-fmc/03a.png)

<br>

Next on Internal Certs, import the Certificate and Private Key for FMC

![x](/static/2024-11-17-ise-trustsec-fmc/04.png)

![x](/static/2024-11-17-ise-trustsec-fmc/05.png)

<br>

## PxGrid Integration

On FMC >> Integration >> Identitiy Sources, select ISE and fill in the details

![x](/static/2024-11-17-ise-trustsec-fmc/06.png)

<br>

When running the Test, we'll get an error saying its in pending state

![x](/static/2024-11-17-ise-trustsec-fmc/07.png)

<br>

Go to ISE >> Administration >> Client Management >> PxGrid Clients, hit approve on the pending client

![x](/static/2024-11-17-ise-trustsec-fmc/08.png)

<br>

Now bac on FMC, redoing the test will result in a success

![x](/static/2024-11-17-ise-trustsec-fmc/09.png)

<br>

## Configuring SGT on ACP

After the integration is complete, we can now use the SGT on the Access Control Policy

![x](/static/2024-11-17-ise-trustsec-fmc/10.png)

<br>

Here we create two rules, SGT_X can access anything on internet while SGT_Y can access 4.4.4.2

![x](/static/2024-11-17-ise-trustsec-fmc/11.png)

<br>

## Dynamic SGT

When an endpoint accesses the network, ISE will dynamically assign an SGT based on the Policy Sets rule, in this case the endpoint got SGT_X

![x](/static/2024-11-17-ise-trustsec-fmc/12.png)

<br>

Therefore its allowed to access internet with no restriction

![x](/static/2024-11-17-ise-trustsec-fmc/13.png)

<br>

## Static SGT

Beside getting SGT mapped from Policy Sets, we can also statically assign an SGT to an IP Address on ISE

![x](/static/2024-11-17-ise-trustsec-asa/17.png)

<br>

Here the endpoint with SGT_Y can only access 4.4.4.2

![x](/static/2024-11-17-ise-trustsec-fmc/14.png)

<br>

And here's the connection events on FMC

![x](/static/2024-11-17-ise-trustsec-fmc/15.png)

<br>














