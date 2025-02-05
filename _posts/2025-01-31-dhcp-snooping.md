---
title: DHCP Snooping & Dynamic ARP Inspection
date: 2025-01-31 11:30:00 +0700
categories: [Networking, DHCP Snooping]
tags: []
---


DHCP Snooping is a security feature that prevents rogue DHCP servers from distributing malicious IP configurations. It works by allowing DHCP responses only from trusted ports and maintaining a binding table of valid IP-MAC pairs.

Dynamic ARP Inspection (DAI) prevents ARP spoofing attacks by verifying ARP packets against the DHCP Snooping binding table. It ensures that only legitimate ARP requests and replies are forwarded, blocking malicious attempts to intercept network traffic.


<br>

## Verifying everything is working

First we'll create the DHCP Pool on Core Switch

![x](/static/2025-01-31-dhcp-snooping/01.png)

```text
ip dhcp pool VLAN20
 network 10.20.0.0 /24
 default-router 10.20.0.1

interface vlan 20
 ip address 10.20.0.1 255.255.255.0

interface Gig0/1
 switchport mode access
 switchport access vlan 20

ip dhcp relay information trust-all
```

<br>

Then on SW-2, we'll put all the ports on VLAN 20 so it can communicate with the Core Switch, and verify the Clients are getting IP Addresses

![x](/static/2025-01-31-dhcp-snooping/02.png)

```text
interface range g0/0-3
 switchport mode access
 switchport access vlan 20

interface range g1/0-3
 switchport mode access
 switchport access vlan 20
```

<br>

Verify Core is giving out IP Addresses with "show ip dhcp binding"

![x](/static/2025-01-31-dhcp-snooping/03.png)

<br>

## Enabling DHCP Snooping

To enable DHCP Snooping on SW-2, run these commands, and then set G0/0 to be trusted interface


![x](/static/2025-01-31-dhcp-snooping/04.png)

```text
ip dhcp snooping
ip dhcp snooping vlan 20
ip dhcp snooping database flash:snooping.db

interface g0/0
 ip dhcp snooping trust
```

<br>

And lastly, we need to run this command if the dhcp server is a cisco network device

```text
ip dhcp relay information trust-all
```

![x](/static/2025-01-31-dhcp-snooping/06.png)

<br>


Run "show ip dhcp snooping" to verify configuration

![x](/static/2025-01-31-dhcp-snooping/05.png)

<br>

And run "show ip dhcp snooping binding" to see the binding records

![x](/static/2025-01-31-dhcp-snooping/08.png)

<br>

## Enabling Dynamic ARP Inspection

To enable DAI, run these commands on SW-2

![x](/static/2025-01-31-dhcp-snooping/09.png)

```
ip arp inspection vlan 20

interface g0/0
 ip arp inspection trust
```

<br>

Run these commands to verify DAI configurations

![x](/static/2025-01-31-dhcp-snooping/10.png)

<br>

And now only Clients with existing binding records will be allowed to access the network

![x](/static/2025-01-31-dhcp-snooping/11.png)

<br>

If we try accessing network with statically configured IP Address, we will be denied access

![x](/static/2025-01-31-dhcp-snooping/12.png)

<br>

And running "show ip arp inspection statistics" will show the drops counter going up

![x](/static/2025-01-31-dhcp-snooping/13.png)

<br>

To add a statically configured IP Address, we can whitelist them with ARP ACL

![x](/static/2025-01-31-dhcp-snooping/14.png)

```
arp access-list ALLOWED-ARP
 permit ip host 10.20.0.250 mac host 52:54:00:06:54:86

ip arp inspection filter ALLOWED-ARP vlan 20
```

<br>

And now on top of the DHCP Snooping table, access are also given to the statically configured MAC Addresses on the ARP ACL

![x](/static/2025-01-31-dhcp-snooping/15.png)


<br>












