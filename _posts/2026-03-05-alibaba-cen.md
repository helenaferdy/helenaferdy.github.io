---
title: Alibaba Cloud Enterprise Network (CEN)
date: 2026-03-05 07:30:00 +0700
categories: [Cloud, Alibaba Cloud]
tags: [Alibaba]
---

Cloud Enterprise Network (CEN) is a high-performance networking service that provides scale-out connectivity to Alibaba Cloud’s global backbone. At the heart of this architecture is the Transit Router (TR), an Enterprise-grade managed hub that acts as the intelligent transit point for all connected VPC spokes and security services.

In this lab, we leverage this hub-and-spoke relationship to build a centralized security architecture. By connecting multiple application VPCs to the Transit Router, we transform our FortiGate VM into a centralized Egress Firewall.

![x](/static/2026-03-05-alibaba-cen/00.png)

<br>

## VPC

Here we are continuing from this [Albaba Cloud Setup](https://senaperdiana.com/posts/alibaba/) lab, where we already have Fortigate and VPC Helena configured

![x](/static/2026-03-05-alibaba-cen/01.png)

<br>

CEN requires us to have multi-zone redundnacy network, so for our helena-vpc we will create additional dummy vs on different zone to satisfy this requirement

![x](/static/2026-03-05-alibaba-cen/02.png)

<br>

Next we create the spokes VPC, starting with a-vpc on 10.10.0.0/24 along with its dummy vs

![x](/static/2026-03-05-alibaba-cen/03.png)

<br>

Same goes for vpc-b on 10.20.0.0/24

![x](/static/2026-03-05-alibaba-cen/04.png)

<br>

And here we have our 3 VPCs

![x](/static/2026-03-05-alibaba-cen/05.png)

<br>

## CEN

Next we configure the Cloud Enterprise Network Instance

![x](/static/2026-03-05-alibaba-cen/06.png)

Here we create the CEN instance named helena-cen

![x](/static/2026-03-05-alibaba-cen/07.png)

![x](/static/2026-03-05-alibaba-cen/08.png)

<br>

Open the helena-cen instance and create a Transit Router

![x](/static/2026-03-05-alibaba-cen/09.png)

<br>

We name it helena-tr

![x](/static/2026-03-05-alibaba-cen/10.png)

![x](/static/2026-03-05-alibaba-cen/11.png)

<br>

Open the Transit Router and here we will create the Intra-region connections to our 3 VPCs

![x](/static/2026-03-05-alibaba-cen/12.png)

<br>

The first one is to our helena-vpc, here we select the int-vs along with its dummy

![x](/static/2026-03-05-alibaba-cen/13.png)

<br>

Then the a-vpc

![x](/static/2026-03-05-alibaba-cen/14.png)

<br>

And the b-vpc

![x](/static/2026-03-05-alibaba-cen/15.png)

<br>

Now we have all connections to the 3 VPCs set up

![x](/static/2026-03-05-alibaba-cen/16.png)

<br>

Next open the Transit Router's Route Tables, here we can see all the routes going to each VPCs are already configured

![x](/static/2026-03-05-alibaba-cen/17.png)

<br>

Add new route, this route will route all default traffic to our helena-vpc

![x](/static/2026-03-05-alibaba-cen/18.png)

![x](/static/2026-03-05-alibaba-cen/19.png)

<br>

## VPC

Now we go back to our VPCs configurations, we will add default routes on each vSwitch level so they all know to route traffic to our Transit Router

![x](/static/2026-03-05-alibaba-cen/20.png)

<br>

On a-vpc & b-vpc, we add a default route that sends traffic to our Transit Router

![x](/static/2026-03-05-alibaba-cen/21.png)

<br>

And on our helena-vpc's int-vs, we add default route to send traffic to fortigate's internal ENI

![x](/static/2026-03-05-alibaba-cen/22.png)

<br>

## ECS

Next we spin up small linux vm on each a-vpc & b-vpc

![x](/static/2026-03-05-alibaba-cen/23.png)

<br>

On the client on a-vpc, we can see that we can connect to client on b-vpc and to internet

![x](/static/2026-03-05-alibaba-cen/24.png)

<br>

Same goes if the test is done from the client on the vpc-b side

![x](/static/2026-03-05-alibaba-cen/25.png)

<br>

On fortigate, logs from both clients can be seen here, verifying that our CEN lab is working and all egress internet traffic is passed through our Fortigate firewall

![x](/static/2026-03-05-alibaba-cen/26.png)

<br>






