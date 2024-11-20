---
title: Cisco ISE TrustSec
date: 2024-11-16 09:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE, RADIUS]
---

Cisco TrustSec (CTS) is a security architecture that simplifies policy enforcement in network environments by using software-defined segmentation. It employs **Security Group Tags (SGTs)** to classify network traffic dynamically, enabling granular access control without relying on complex IP-based ACLs. TrustSec uses **Cisco TrustSec-capable devices** to propagate SGTs throughout the network, ensuring policy consistency. 

SGT Exchange Protocol (SXP) is a key component that allows devices incapable of assigning SGTs to participate in the TrustSec architecture by mapping IP addresses to SGTs and sharing this mapping with other TrustSec-enabled devices. This enables a unified and scalable approach to network segmentation and access control.


## Configuring TrustSec on ISE

On ISE, add the NAD and enable TrustSec along with the usual Radius configuration, this device will become the Enforcement Point for the TrustSec rules

![x](/static/2024-11-16-ise-trustsec/01.png)

![x](/static/2024-11-16-ise-trustsec/02.png)

![x](/static/2024-11-16-ise-trustsec/03.png)

<br>

Next also add the NAD as an SXP Listener Device so it will get updates about our SGT Mappings

![x](/static/2024-11-16-ise-trustsec/04.png)

> Security Group Tag Exchange Protocol (SXP) is a protocol used in Cisco TrustSec to propagate SGT bindings across the network.

<br>

After that create some SGTs, here we'll use SGT_X, SGT_Y, and SGT_Z

![x](/static/2024-11-16-ise-trustsec/05.png)

<br>

Next create two SGACL, one for permit access and the other for deny access

![x](/static/2024-11-16-ise-trustsec/06.png)

> * SGACL (Security Group Access Control List) is a policy enforcement mechanism that applies access control rules based on SGTs rather than IP addresses.

<br>

After that define the SGT access policy by creating the SGT Matrix, here we enable access from SGT_X to SGT_Y, but deny it to SGT_Z

![x](/static/2024-11-16-ise-trustsec/07.png)

> * The SGT Matrix is a policy table that maps interactions between source and destination SGTs, specifying the level of access allowed using SGACL.

<br>

Next we configure the Policy Sets to dynamically assigns SGT to endpoint

![x](/static/2024-11-16-ise-trustsec/09.png)

<br>

Or we can statically assign the SGT to specific IP Addresses on ISE

![x](/static/2024-11-16-ise-trustsec/08.png)

<br>

## Configuring TrustSec on NAD

Before configuring TrustSec, make sure the NAD alrady has radius configuration for dot1x/mab authentication, after then we can put these config on top of it.

![x](/static/2024-11-16-ise-trustsec/12.png)

```
cts sxp enable
cts sxp default password helena  
cts sxp default source-ip 
cts sxp connection peer 198.18.128.11 password default mode local listener
cts role-based enforcement

aaa authorization network CTS group ISE
cts authorization list CTS
cts credentials id SW1 password helena

radius server ISE1
 address ipv4 198.18.128.11 auth-port 1812 acct-port 1813
  pac key helena
```

<br>

After config has been applied, NAD will start doing CTS authentication that we can see on Radius Logs

![x](/static/2024-11-16-ise-trustsec/13.png)

<br>

Running "show cts pacs" confirms this process has been complete

![x](/static/2024-11-16-ise-trustsec/14.png)

> PACs (Privilege Authorization Certificates) are secure, cryptographic credentials used in Cisco TrustSec to authenticate and establish trust between network devices, enabling secure propagation of Security Group Tags (SGTs).

<br>

Command "show cts environment-data" shows the NAD has successfully downloaded the environment-data which contains the SGTs

![x](/static/2024-11-16-ise-trustsec/15.png)

<br>

Running "show cts role-based permission" shows SGT Matrix as rbac rules, this rules are also automatically applied as access-lists

![x](/static/2024-11-16-ise-trustsec/16.png)

<br>

The command "show cts role-based sgt-map all" shows the IP-to-SGT mappings which are obtained from internal sessions or from SXP

![x](/static/2024-11-16-ise-trustsec/17.png)

<br>

Talking about SXP, run "show cts sxp connections" to validate SXP connection to ISE

![x](/static/2024-11-16-ise-trustsec/18.png)

<br>

The SXP status should also be reflected on ISE

![x](/static/2024-11-16-ise-trustsec/19.png)

<br>

To show all SGT Mappings obtained throgh SXP, run "show cts sxp sgt-map"

![x](/static/2024-11-16-ise-trustsec/19a.png)

<br>

## Testing TrustSec

Here we have devices all in the same subnet, where .100 with SGT_X can access .101 with SGT_Y just fine, but not able to access .102 with SGT_Z

![x](/static/2024-11-16-ise-trustsec/20.png)

<br>

## Modifying SGT Matrix

Now lets add a new SGACL to block traffic on port 443

![x](/static/2024-11-16-ise-trustsec/21.png)

<br>

Here we put the SGACL on matrix from SGT_X to SGT_Z

![x](/static/2024-11-16-ise-trustsec/22.png)

<br>

Run "cts refresh policy" to force the update on the NAD side

![x](/static/2024-11-16-ise-trustsec/23.png)

<br>

And now the .100 client can still ping or access any port on .102, except for port 443















