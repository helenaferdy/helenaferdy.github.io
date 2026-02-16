---
title: Azure Application Gateway with WAF
date: 2026-02-16 07:30:00 +0700
categories: [Cloud, Azure]
tags: [Azure]
---

Azure Application Gateway is a Layer 7 (HTTP/HTTPS) load balancer that sits at the front of our application, accepts internet traffic on a public IP, and routes it to backend servers based on listener and routing rules. The Web Application Firewall (WAF) adds security by inspecting requests for common web attacks (SQL injection, XSS, OWASP threats) before forwarding clean traffic to the backend pool.

![x](/static/2026-02-16-azure-appgw/00.png)

<br>

## Application Gateway

Here we have 2 linux servers on the internal network hosting web on port 8080

![x](/static/2026-02-16-azure-appgw/01.png)

<br>

Next on the Application Gateways, create new

![x](/static/2026-02-16-azure-appgw/02.png)

<br>

Here we will select Tier Standard first, before changing it later to enable WAF. We also place this App Gateway on the external subnet

![x](/static/2026-02-16-azure-appgw/03.png)

<br>

For the frontend, give it a Public IP Address

![x](/static/2026-02-16-azure-appgw/04.png)

<br>

For the backend, point it to both our linux servers

![x](/static/2026-02-16-azure-appgw/05.png)

<br>

Next add a routing rule that listens on port 80 on frontend

![x](/static/2026-02-16-azure-appgw/06.png)

<br>

And route it to port 8080 on the backend

![x](/static/2026-02-16-azure-appgw/07.png)

![x](/static/2026-02-16-azure-appgw/08.png)

<br>

That should finish our Application Gateway configuration

![x](/static/2026-02-16-azure-appgw/09.png)

![x](/static/2026-02-16-azure-appgw/10.png)

<br>

We can open the AppGW to see the health of the backend members

![x](/static/2026-02-16-azure-appgw/11.png)

<br>

To test this, we visit the public IP on port 80, the traffic should be load balanced to our 2 linux servers

![x](/static/2026-02-16-azure-appgw/12.png)

![x](/static/2026-02-16-azure-appgw/13.png)

<br>

## WAF

To enable WAF, we change the tier from Standard to WAF	

![x](/static/2026-02-16-azure-appgw/14.png)

<br>

Then on Network Security, open Web Application Firewalls and create new

![x](/static/2026-02-16-azure-appgw/15.png)

<br>

Select regional WAF and use Prevention mode

![x](/static/2026-02-16-azure-appgw/16.png)

<br>

Next we will skip the default policies and create our own custom ones, the first policy is to block traffic from China

![x](/static/2026-02-16-azure-appgw/17.png)

<br>

And the second one is to throttle limit connections

![x](/static/2026-02-16-azure-appgw/18.png)

<br>

These 2 rules should be enough for this lab test

![x](/static/2026-02-16-azure-appgw/19.png)

<br>

Next associate the WAF to our App Gateway

![x](/static/2026-02-16-azure-appgw/20.png)

<br>

And that should finish our WAF configuration

![x](/static/2026-02-16-azure-appgw/21.png)

<br>

Because we associated it to our AppGW, when we open the AppGW it shows that our WAF is connected

![x](/static/2026-02-16-azure-appgw/20a.png)

<br>

Now when we try accessing our web using China based VPN or when we try to brute force it, we should be denied

![x](/static/2026-02-16-azure-appgw/22.png)

<br>























