---
title: Fortinet Security Fabric
date: 2026-02-08 07:00:00 +0700
categories: [Security, Fortinet]
tags: [Fortinet]
---


The Fortinet Security Fabric is an integrated architecture that enables multiple security devices to share telemetry and coordinate a unified response to threats. This lab demonstrates the seamless integration of FortiGates with FortiManager and FortiAnalyzer, consolidating individual appliances into a unified, centrally managed security ecosystem

![x](/static/2026-02-08-forti-security-fabric/00.png)

<br>

## Security Fabric

Here we have Fortigate-14 which will act as our Fabric Root that already has connection to Fortianalyzer

![x](/static/2026-02-08-forti-security-fabric/01.png)

![x](/static/2026-02-08-forti-security-fabric/02.png)

<br>

Here we enable Security Fabric as Root

![x](/static/2026-02-08-forti-security-fabric/03.png)

<br>

After that we can move to our members, in this case is the Fortigate-15. Here we also enable Security Fabric as a member

![x](/static/2026-02-08-forti-security-fabric/04.png)

<br>

Do the same for Fortigate-16, and now we have both members show up on Fabric Root pending authorization

![x](/static/2026-02-08-forti-security-fabric/05.png)

<br>

Authorize both members

![x](/static/2026-02-08-forti-security-fabric/06.png)

<br>

And now we have both members as part of the Security Fabric

![x](/static/2026-02-08-forti-security-fabric/07.png)

<br>

Because Fortigate-14 alread has Fortianalyzer configured, this configuration will be synced to the downstream members, automatically adding them to the same FAZ. All members now show up on FAZ as Unauthorized Devices which we just need to authorize

![x](/static/2026-02-08-forti-security-fabric/08.png)

![x](/static/2026-02-08-forti-security-fabric/09.png)

<br>

Here's how the FAZ config looks on the downstream member, we can't modify the config because it's synced from the fabric root

![x](/static/2026-02-08-forti-security-fabric/10.png)

<br>

## Object Synchronization

One of the cool things about Security Fabric is we can create a firewall object on the Fabric Root and it will be autmatically propagated to all downstream members

![x](/static/2026-02-08-forti-security-fabric/11.png)

> Object on Fortigate-14

![x](/static/2026-02-08-forti-security-fabric/12.png)

> Object on Fortigate-15

<br>

## FortiManager

Next let's add the Fortigate-14 to FMG

![x](/static/2026-02-08-forti-security-fabric/13.png)

<br>

Once the Fabric Root is registered, all members will also show up on FMG as Unauthorized

![x](/static/2026-02-08-forti-security-fabric/14.png)

<br>

Lets authorize the devices

![x](/static/2026-02-08-forti-security-fabric/15.png)

![x](/static/2026-02-08-forti-security-fabric/16.png)

<br>

On the member, we can see the FMG configuration is synced from the Fabric Root

![x](/static/2026-02-08-forti-security-fabric/17.png)

![x](/static/2026-02-08-forti-security-fabric/18.png)

<br>




