---
title: Cisco ISE Dot1x & MAB
date: 2024-06-16 10:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE]
---

Cisco Identity Services Engine (ISE) :
> * Dot1x (802.1x) is a network access control protocol that uses EAP (Extensible Authentication Protocol) to authenticate devices at the network edge by verifying user credentials against a RADIUS server before granting access. 
> * MAB (MAC Authentication Bypass) is a fallback mechanism for non-802.1x devices, using the device's MAC address to authenticate and grant network access via a RADIUS server when 802.1x is not supported.


Here's the topology for this lab

![x](/static/2024-06-16-ise-dot1x-mab/00.png)

<br>

## Preparation

First on Administration >> Network Devices, add the switch-1 as a Network Device with RADIUS Authentication enabled

![x](/static/2024-06-16-ise-dot1x-mab/01.png)

<br>

Next on Administration >> Identities, create an Internal User used for dot1x authentication

![x](/static/2024-06-16-ise-dot1x-mab/02.png)

<br>

After that, on Work Center >> Guest Access >> Identities, add the client's MAC Address, this will be used for MAB authentication

![x](/static/2024-06-16-ise-dot1x-mab/06.png)

![x](/static/2024-06-16-ise-dot1x-mab/07.png)

<br>

The added MAC Address can be found on Identity Groups

![x](/static/2024-06-16-ise-dot1x-mab/08.png)

<br>

## Configuring MAB

On Policy >> Policy Sets, create a new Policy Sets that captures all RADIUS authentication

![x](/static/2024-06-16-ise-dot1x-mab/02a.png)

<br>

Next on the Authentication Policy, create a rule to capture all MAB authentication

![x](/static/2024-06-16-ise-dot1x-mab/02b.png)

<br>

Then on Authorization Policy, add a rule to permit network access if authentication passed

![x](/static/2024-06-16-ise-dot1x-mab/04.png)

<br>

### MAB Configuration on NAD

Configure the switch to use MAB as the primary order of authentication

![x](/static/2024-06-16-ise-dot1x-mab/09.png)

```text
aaa new-model
aaa authentication dot1x default group radius
radius server host
address ipv4 198.18.134.31
key helena
exit

dot1x system-auth-control

int eth0/2
switchport host
authentication port-control auto
authentication order mab dot1x
dot1x pae authe
mab
exit
```

<br>

Test the radius server to make sure the connection is up and running

![x](/static/2024-06-16-ise-dot1x-mab/10.png)

<br>

The successful test log can be seen on ISE

![x](/static/2024-06-16-ise-dot1x-mab/11.png)

<br>

### MAB Testing

On the Client PC, enable the Wired Autoconfig service

![x](/static/2024-06-16-ise-dot1x-mab/12.png)

<br>

And on the Switch, we can see the client has successfully done the MAB authentication 

![x](/static/2024-06-16-ise-dot1x-mab/13.png)

<br>

And the RADIUS Live Logs shows the successful auth

![x](/static/2024-06-16-ise-dot1x-mab/14.png)

<br>

And here's the detail of the logs

![x](/static/2024-06-16-ise-dot1x-mab/14a.png)

<br>

## Configuring Dot1x

On the Policy Sets, lets add a new authentication rule for Dot1x

![x](/static/2024-06-16-ise-dot1x-mab/03.png)

<br>

And on the switch, switch the order so dot1x is used first

![x](/static/2024-06-16-ise-dot1x-mab/14b.png)

<br>

### Dot1x Testing

On Client PC, configure the Dot1x Authentication on the NIC Adapter by supplying username and password

![x](/static/2024-06-16-ise-dot1x-mab/14c.png)

<br>

And after the pc successfully authenticated, we can see on the switch the method has changed to dot1x

![x](/static/2024-06-16-ise-dot1x-mab/15.png)

<br>

The RADIUS Live Logs shows the successful dot1x authentication

![x](/static/2024-06-16-ise-dot1x-mab/16.png)

![x](/static/2024-06-16-ise-dot1x-mab/16a.png)

<br>

### Testing MAB Failover

Now lets test the MAB failover if the dot1x fails, to do that we'll supply an incorrect credential

![x](/static/2024-06-16-ise-dot1x-mab/17.png)

<br>

Here on RADIUS Live Logs we can see that it tried Dot1x first but failed, then proceed with MAB

![x](/static/2024-06-16-ise-dot1x-mab/18.png)

<br>

On the client, even though the Dot1x Authentication failed we can still access the network thanks to MAB

![x](/static/2024-06-16-ise-dot1x-mab/19.png)

<br>











































