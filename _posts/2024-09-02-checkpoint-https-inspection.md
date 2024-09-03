---
title: Check Point HTTPS Inspection
date: 2024-09-02 01:00:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point HTTPS Inspection is a security feature that allows Check Point firewalls to inspect encrypted HTTPS traffic for potential threats. By decrypting SSL/TLS traffic, the firewall can analyze the data for malicious content, apply security policies, and then re-encrypt the traffic before forwarding it to its destination. This process helps protect against encrypted malware, data breaches, and other threats that might otherwise bypass security controls.

## Dealing with Certificates

First lets create a CA Signed Certificate for the Check Point Security Gateway, here we'll use certtools to create the CSR

![x](/static/2024-09-02-checkpoint-https-inspection/01.png)

<br>

On Key Usage, we'll select Digital Signature, Certificate Sign, and CRL Sign to create a Subordinate CA Certificate. This is used for HTTPS inspection on a firewall to enable the firewall to act as a trusted intermediary between clients and servers. When the firewall intercepts HTTPS traffic for inspection, it decrypts the traffic using a subordinate CA certificate it has generated or been assigned. 

![x](/static/2024-09-02-checkpoint-https-inspection/02.png)

> 1. Digital Signature: <br>
> Needed. This is essential for creating digital signatures on certificates issued by the subordinate CA. Digital signatures ensure the authenticity and integrity of the certificates. <br>
> 2. Certificate Sign: <br>
> Needed. This allows the subordinate CA to sign certificates. For HTTPS inspection, the firewall will dynamically generate and sign certificates for inspected HTTPS traffic, mimicking the original server certificates. <br>
> 3. CRL Sign: <br>
> Optional but Recommended. This allows the subordinate CA to sign Certificate Revocation Lists (CRLs), which are used to revoke certificates that should no longer be trusted. Including this can help maintain the security of the HTTPS inspection environment. <br>

<br>

Next we sign this CSR on our CA Server using Subordinate CA Template

![x](/static/2024-09-02-checkpoint-https-inspection/03.png)

<br>

Now we have the Certificate and Key file, in order to use these on Check Point, we need to convert them into a .p12 file using openssl

```shell
openssl pkcs12 -export -out cpsg.p12 -inkey cpsg.key -in cpsg.cer -certfile cpsg.cer -legacy
```

![x](/static/2024-09-02-checkpoint-https-inspection/04.png)

> * .cer (Certificate): This file format typically contains only the public key certificate, which includes information about the certificate holder, the issuing CA, and the public key.
> * .key (Private Key): This file contains the private key associated with a public key certificate. The .key file is used to sign data and decrypt information encrypted with the corresponding public key.
> * .p12 (PKCS#12 or PFX): This is a binary file format that can store a certificate along with its private key and any intermediate certificates in a single, encrypted file.

<br>

Next open Smart Dashboard >> HTTPS Inspection >> Trusted CA, here we import the Root CA

![x](/static/2024-09-02-checkpoint-https-inspection/05.png)

<br>

Then on Gateways, select Import Outbond Certificate and import the .p12 file

![x](/static/2024-09-02-checkpoint-https-inspection/06.png)

<br>

Now the ceritificate has been successfully imported

![x](/static/2024-09-02-checkpoint-https-inspection/07.png)

<br>

## Enabling HTTPS Inspection

Now back to Smart Console, enable the HTTPS Inspection on the Security Gateway

![x](/static/2024-09-02-checkpoint-https-inspection/08.png)

<br>

Next on Security Policies >> Policy >> Gateway, edit the assigned policy to be Recommended Inspection

![x](/static/2024-09-02-checkpoint-https-inspection/09.png)

<br>

Then on Policy, here we configure the policy to define the scope of traffic that will be inspected

![x](/static/2024-09-02-checkpoint-https-inspection/10.png)

<br>

After publishing and installing, now all HTTPS traffic from the client nodes will be inspected, as showm below

![x](/static/2024-09-02-checkpoint-https-inspection/11.png)

![x](/static/2024-09-02-checkpoint-https-inspection/12.png)

<br>

We can also see the logs of the traffic being inspected

![x](/static/2024-09-02-checkpoint-https-inspection/13.png)

![x](/static/2024-09-02-checkpoint-https-inspection/14.png)

<br>

On Smart Dashboard, we can modify the HTTPS Validation, for example here we wanted to make it to be more strict by dropping traffic from lists selected below

![x](/static/2024-09-02-checkpoint-https-inspection/15.png)

<br>

And now if we try accessing sites with bad certificates, the traffic will be rejected

![x](/static/2024-09-02-checkpoint-https-inspection/16.png)

<br>
























