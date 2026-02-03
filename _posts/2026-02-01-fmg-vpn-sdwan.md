---
title: FortiManager VPN & SD-WAN Manager
date: 2026-02-01 07:00:00 +0700
categories: [Security, Fortinet]
tags: [Fortinet]
---

In this lab we will deploy Fortinet's Hub-and-spokes VPN SD-WAN solution with VPN Manager and SD-WAN Manager inside the FortiManager following this topology

![x](/static/2026-02-01-fmg-vpn-sdwan/00.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/01.png)

<br>

## VPN Manager

VPN Managaer is a centralized tool in FortiManager 7.6 to design, configure, and deploy IPsec and SSL VPNs across multiple FortiGate devices using templates and topology-based workflows. <br>
First we will create a VPN Community with Remote Access type for Dial-Up VPN.

![x](/static/2026-02-01-fmg-vpn-sdwan/02.png)

<br>

Then we configure the Phase 1 and Phase 2

![x](/static/2026-02-01-fmg-vpn-sdwan/03.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/04.png)

<br>

Review the configs and hit OK

![x](/static/2026-02-01-fmg-vpn-sdwan/05.png)

<br>

Next right click on the community and select Add Managed Gateway, here we will add the first device which is our HUB

![x](/static/2026-02-01-fmg-vpn-sdwan/06.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/07.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/08.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/09.png)

<br>

On the last step, here we configure so branches that connect to this VPN will be automatically assigned Overlay IP Address

![x](/static/2026-02-01-fmg-vpn-sdwan/10.png)

<br>

Next we will add the Branch

![x](/static/2026-02-01-fmg-vpn-sdwan/11.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/12.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/13.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/14.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/15.png)

<br>

We will also use the Provisioning Template to help up configure some additional settings, first one is the static route for the Overlay subnets on the VPN Interface

![x](/static/2026-02-01-fmg-vpn-sdwan/16.png)

<br>

Then we also need the BGP configurations for Hub

![x](/static/2026-02-01-fmg-vpn-sdwan/17.png)

<br>

And also the spokes

![x](/static/2026-02-01-fmg-vpn-sdwan/18.png)

<br>

Hub's VPN Interface will not get IP from DHCP, so having a CLI Template to configure that automatically is a nice thing to have

![x](/static/2026-02-01-fmg-vpn-sdwan/19.png)

<br>

While we're at it, lets also add some firewall policies using Policy Package

![x](/static/2026-02-01-fmg-vpn-sdwan/20.png)

<br>

After all's done, now we can push the configurations

![x](/static/2026-02-01-fmg-vpn-sdwan/21.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/22.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/23.png)

<br>

And just like that, now we have a working Hub-and-spokes Dial-up VPN for all of our managed devices

![x](/static/2026-02-01-fmg-vpn-sdwan/24.png)

<br>

## SD-WAN Manager

While VPN Manager focuses only VPN (hence the name), if we want to create a complete SD-WAN solution its better to use the SD-WAN Manager. <br>
To use it, we first need to enable Managed by SD-WAN Manager toggle for our devices. This option makes devices no longer configurable though Device Manager.

![x](/static/2026-02-01-fmg-vpn-sdwan/25.png)

<br>

Instead we now manage our devices using SD-WAN Manager

![x](/static/2026-02-01-fmg-vpn-sdwan/26.png)

<br>

We are required to put our branches devices into its own group so lets configure that first

![x](/static/2026-02-01-fmg-vpn-sdwan/27.png)

<br>

Now we will make a Provisioning Template that's tailored specifically for SD-WAN deployment. To do that we will use the Overlay Orchestration and create a new template. <br>
We select the 1 Hub type, enter the Loopback and Overlay networks

![x](/static/2026-02-01-fmg-vpn-sdwan/28.png)

<br>

Then we select the devices that will become Hub and Spokes

![x](/static/2026-02-01-fmg-vpn-sdwan/29.png)

<br>

Then we assign the interfaces for the VPN Tunnel and for the Inside Network for each site

![x](/static/2026-02-01-fmg-vpn-sdwan/30.png)

<br>

Crucially here we add the static route for Overlay Networks so all Hub and Spokes can reach each other through the VPN

![x](/static/2026-02-01-fmg-vpn-sdwan/31.png)

<br>

Review and hit Finish

![x](/static/2026-02-01-fmg-vpn-sdwan/32.png)

<br>

Now we have the Provisioning Templates for Hub and Spokes that will configure the VPN, Static Route, and BGP

![x](/static/2026-02-01-fmg-vpn-sdwan/33.png)

<br>

Lets push the configs

![x](/static/2026-02-01-fmg-vpn-sdwan/34.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/35.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/36.png)

<br>

And just like that now we have a working Hub-and-spoke Dial-up VPN with internal BGP

![x](/static/2026-02-01-fmg-vpn-sdwan/37.png)

<br>

One this is still missing, which is the SD-WAN configuration. Here we will create a new SD-WAN Template for Hub and Spokes

![x](/static/2026-02-01-fmg-vpn-sdwan/38.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/39.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/40.png)

<br>

Now we can add the SD-WAN templates into our Template Groups as well, completing the SD-WAN solution configuration

![x](/static/2026-02-01-fmg-vpn-sdwan/41.png)

<br>

Next we push the configs

![x](/static/2026-02-01-fmg-vpn-sdwan/42.png)

![x](/static/2026-02-01-fmg-vpn-sdwan/43.png)

<br>

And lastly we now also have the SD-WAN configuration up and running on all of our managed devices

![x](/static/2026-02-01-fmg-vpn-sdwan/44.png)

<br>

That concludes our SD-WAN configuration using SD-WAN Manager

![x](/static/2026-02-01-fmg-vpn-sdwan/45.png)

<br>



