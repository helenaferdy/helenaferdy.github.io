---
title: Alibaba Application Load Balancer with WAF
date: 2026-04-04 07:30:00 +0700
categories: [Cloud, Alibaba Cloud]
tags: [Alibaba]
---

Alibaba Cloud ALB distributes incoming internet traffic across multiple backend servers to ensure high availability. When integrated with WAF, it filters and protects traffic (e.g., rate limiting, geo-filtering) before forwarding only clean HTTP requests to the backend servers.

![x](/static/2026-04-04-alibaba-alb/00.png)

<br>


## VPC

Before configuring ALB, we need to ensure that our External vSwitch has multi domain availability, here we create a Dummy vSwitch on a different zone to fulfill that requirement

![x](/static/2026-04-04-alibaba-alb/00a.png)

<br>


## Instances

For the instances, we have 2 linux servers serving web on port 8080

![x](/static/2026-04-04-alibaba-alb/01.png)

![x](/static/2026-04-04-alibaba-alb/02.png)

<br>

## ALB

Next we go to Server Load Balancer and select create new on ALB

![x](/static/2026-04-04-alibaba-alb/03.png)

<br>

The network type is Internet and we choose our 2 vswitches on different zones, for the Edition we will configure standard for now and enable WAF later

![x](/static/2026-04-04-alibaba-alb/04.png)

![x](/static/2026-04-04-alibaba-alb/05.png)

<br>

After that we configure the Server Group, create new

![x](/static/2026-04-04-alibaba-alb/06.png)

<br>

We name it Web_Servers and configure Health Check on port 8080

![x](/static/2026-04-04-alibaba-alb/07.png)

<br>

Then add the backend servers, which are our linux servers

![x](/static/2026-04-04-alibaba-alb/08.png)

![x](/static/2026-04-04-alibaba-alb/09.png)

<br>

Back to our ALB Instance, now we configure the Listener

![x](/static/2026-04-04-alibaba-alb/10.png)

<br>

We set the listener to be on port 80 (HTTP) and point it to our Backend Servers

![x](/static/2026-04-04-alibaba-alb/11.png)

![x](/static/2026-04-04-alibaba-alb/12.png)

<br>

That should conclude our ALB configuration, we can see our listener is now up and healthy

![x](/static/2026-04-04-alibaba-alb/14.png)

<br>

And we can copy the Domain Name and the EIP for accessing the ALB from the internet

![x](/static/2026-04-04-alibaba-alb/13.png)

<br>

Accessing the EIP or the Domain Name allows us to reach the ALB which load balances the traffic to both our linux servers

![x](/static/2026-04-04-alibaba-alb/15.png)

<br>

## WAF

Next we enable WAF, to do that open the ALB Instance and hit enable WAF

![x](/static/2026-04-04-alibaba-alb/16.png)

![x](/static/2026-04-04-alibaba-alb/17.png)

<br>

That will change the edition from Standard to WAF Enabled

![x](/static/2026-04-04-alibaba-alb/18.png)

<br>

We can open WAF dedicated console and on the Protected Objects we will see our ALB Instance

![x](/static/2026-04-04-alibaba-alb/19.png)

<br>

On Core Web Protection, we can configure rules that's applied on the WAF, here we will configure geo-blocking and rate-limiting

![x](/static/2026-04-04-alibaba-alb/20.png)

<br>

First the Geo-blocking, we will block any traffic coming in from Russia

![x](/static/2026-04-04-alibaba-alb/21.png)

<br>

Then Rate-limiting, create new Custom Rule add a Rate Limiting Protection

![x](/static/2026-04-04-alibaba-alb/22.png)

![x](/static/2026-04-04-alibaba-alb/23.png)

<br>

Now the Core Web Protection shows that Custom Rule and Geo-Blocking has active rules

![x](/static/2026-04-04-alibaba-alb/24.png)

<br>

And finally if we try accessing it from Russia or spamming the url, we will get blocked

![x](/static/2026-04-04-alibaba-alb/25.png)

<br>

























