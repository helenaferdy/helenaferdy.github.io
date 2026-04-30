---
title: Cisco ThousandEyes
date: 2026-04-28 07:30:00 +0700
categories: [Monitoring Tools, ThousandEyes]
tags: [ThousandEyes]
---

Cisco ThousandEyes is a synthetic monitoring platform that acts as a fleet of automated users, giving us hop-by-hop visibility into local network routing, global internet paths, and application performance.

In this lab, we build a comprehensive 360-degree monitoring strategy using three distinct vantage points:
> * Virtual Appliance (TE VA): A dedicated internal VM serving as the baseline monitor for our core network.
> * Catalyst 8000v Enterprise Agent: A containerized probe embedded directly in the edge router.
> * Windows 10 Endpoint Agent: A lightweight background app measuring the true, end-to-end user experience.

![x](/static/2026-04-28-thousandeyes/00.png)

<br>

Cisco allows us to try ThousandEyes for 15 days, we can request it directly at cisco.com

![x](/static/2026-04-28-thousandeyes/01.png)

<br>

TE works by using Agents that periodically perform network testing, so here we will deploy some Enterprise Agents

![x](/static/2026-04-28-thousandeyes/02.png)

<br>

## Appliance Agent

First we will add an Appliance Agent that sits behind our Edge router, here we download the installer and copy the token

![x](/static/2026-04-28-thousandeyes/03.png)

<br>

Then deploy the VM and run the instalation wizard

![x](/static/2026-04-28-thousandeyes/04.png)

<br>

Configure the nettwork settings

![x](/static/2026-04-28-thousandeyes/05.png)

<br>

After that we will be able to access the GUI

![x](/static/2026-04-28-thousandeyes/06.png)

![x](/static/2026-04-28-thousandeyes/07.png)

<br>

Here we enter the token	and finish the configuration

![x](/static/2026-04-28-thousandeyes/08.png)

<br>

Ensure everything is green and we're good to go

![x](/static/2026-04-28-thousandeyes/09.png)

<br>

Back to the TE Portal, we will see our VA being registered in Agent Settings

![x](/static/2026-04-28-thousandeyes/10.png)

![x](/static/2026-04-28-thousandeyes/11.png)

<br>

## Cisco Agent

Next we will add a cisco's built-in agent directly on the Edge Router, the agent requires its own network segment that's being NATted so it can reach the internet

![x](/static/2026-04-28-thousandeyes/12.png)

<br>

First we enable IOx and make sure the service is running

![x](/static/2026-04-28-thousandeyes/14.png)

> IOx is application hosting platform on the router, allowing us to run Linux apps or containers directly on router

<br>

Then we download the agent and copy it over to the router

![x](/static/2026-04-28-thousandeyes/13.png)

<br>

After that install the agent and ensure the state is 'DEPLOYED'

```
app-hosting install appid te package flash:tex.tar
show app-hosting list
```

![x](/static/2026-04-28-thousandeyes/15.png)

<br>

Then we configure the Virtual Interface and NAT

```
interface VirtualPortGroup0
ip address 10.99.0.1 255.255.255.0
ip nat inside

ip access-list standard 1
11 permit 10.99.0.0 0.0.0.255

ip nat inside source list 1 interface GigabitEthernet2 overload
```

![x](/static/2026-04-28-thousandeyes/16.png)

<br>

Then we configure the TE App and Docker, here we also provide the registration token

```
app-hosting appid te
 app-vnic gateway0 virtualportgroup 0 guest-interface 0
  guest-ipaddress 10.99.0.99 netmask 255.255.255.0
  app-default-gateway 10.99.0.1 guest-interface 0
 name-server0 8.8.8.8
 app-resource docker
  prepend-pkg-opts
  run-opts 1 "-e TEAGENT_ACCOUNT_TOKEN=mnvo2hqgvugw3n7tde9r9awwyewkbbgi"
```

![x](/static/2026-04-28-thousandeyes/17.png)

<br>

Finally we can activate and start the TE App

```
app-hosting activate appid te
app-hosting start appid te
```

![x](/static/2026-04-28-thousandeyes/18.png)

<br>

Verify the app is up and running

```
show app-hosting list
show app-hosting detail appid te
```

![x](/static/2026-04-28-thousandeyes/19.png)

![x](/static/2026-04-28-thousandeyes/20.png)

<br>

Back to the TE Portal, we can see now the Edge Router is registered

![x](/static/2026-04-28-thousandeyes/21.png)

![x](/static/2026-04-28-thousandeyes/22.png)

<br>

## Creating Testing

Now that we have our agents running, we can start creating test cases. We will use the Agent to Server test to evaluate our network health to the specified server, which is senaperdiana.com

![x](/static/2026-04-28-thousandeyes/23.png)

![x](/static/2026-04-28-thousandeyes/24.png)

![x](/static/2026-04-28-thousandeyes/25.png)

<br>

After leaving it running for some time, we can see that we're now getting data for network latency as well as path visualization, giving as exact detail of where the latency is actually taking place

![x](/static/2026-04-28-thousandeyes/26.png)

<br>

Here we will simulate a network latency on our Check Point firewall, by disabling acceleration and adding 500ms on its egress interface

```
fwaccel off
tc qdisc add dev eth1 root netem delay 500ms
```

![x](/static/2026-04-28-thousandeyes/27.png)

<br>

And on TE, the exact point where the network went bad will be shown as red

![x](/static/2026-04-28-thousandeyes/28.png)

![x](/static/2026-04-28-thousandeyes/29.png)

![x](/static/2026-04-28-thousandeyes/30.png)

<br>

Other than latency, we can also see packet loss, jitter, and some more

![x](/static/2026-04-28-thousandeyes/31.png)

<br>

## Endpoint Experience

Other than Enterprise Agent, we also have Endpoint Agent where it monitors performance directly from the user's device (like a laptop or desktop) using a lightweight background agent.

![x](/static/2026-04-28-thousandeyes/32.png)

<br>

First we download the agent

![x](/static/2026-04-28-thousandeyes/33.png)

<br>

Then run the installer and enter the token

![x](/static/2026-04-28-thousandeyes/34.png)

<br>

After that finishes, the computers should show up on our listed endpoint agents

![x](/static/2026-04-28-thousandeyes/35.png)

<br>

## Creating User Testing

We basically have 2 testing types here, real test and synthetic test, first one is real test where we configure specified domain that users access through their browser so we can get some telemetry. Here we will configure senaperdiana.com

![x](/static/2026-04-28-thousandeyes/36.png)

<br>

Now when users access senaperdiana.com, the installed extension will capture the data and send it to TE

![x](/static/2026-04-28-thousandeyes/37.png)

<br>

Allowing us to monitor the user experience and the overall network health of that specified user

![x](/static/2026-04-28-thousandeyes/38.png)

<br>

The other test is synthetic test, where we use the built-in templates that run periodically to get the network telemetry

![x](/static/2026-04-28-thousandeyes/38a.png)

<br>

Here we configure the Google Suites template

![x](/static/2026-04-28-thousandeyes/39.png)

<br>

And after it runs for some time, we will start getting data and visualization on our TE Portal

![x](/static/2026-04-28-thousandeyes/40.png)

![x](/static/2026-04-28-thousandeyes/41.png)

![x](/static/2026-04-28-thousandeyes/42.png)

![x](/static/2026-04-28-thousandeyes/43.png)

<br>












