---
title: Check Point Infinity
date: 2026-01-07 07:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point Infinity is a consolidated security architecture that integrates network (Quantum), cloud (CloudGuard), and endpoint/workload (Harmony) security into a single policy, threat-intelligence, and management plane. It leverages shared services such as ThreatCloud, unified logging, and centralized enforcement to provide consistent, real-time threat prevention across on-prem, cloud, and hybrid environments.

<br>

## Enabling Infinity Portal

On Smartconsole, we can enable integration between On-prem Quantum devices with Infinity Portal on the Infinity Services tab and hit Get Started

![x](/static/2026-01-07-cp-infinity/01.png)

<br>

Then input the token from the Infinity Portal

![x](/static/2026-01-07-cp-infinity/04.png)

![x](/static/2026-01-07-cp-infinity/03.png)

<br>

If successful, the status will become active

![x](/static/2026-01-07-cp-infinity/05.png)

<br>

Next select 3 dots on top right corner and enable both Configuration and Log Sharing so Infinity Portal can consume them

![x](/static/2026-01-07-cp-infinity/06.png)

![x](/static/2026-01-07-cp-infinity/07.png)

<br>

On Infinity Portal, we should see our Smart-1 shows up as connected management

![x](/static/2026-01-07-cp-infinity/09.png)

<br>

## AI Copilot

First Infinity Feature that we will try is AI Copilot, it's Check Point’s generative-AI assistant that integrates with the Infinity Platform to help us automate and accelerate tasks like policy management, event analysis, incident response, and threat hunting using chatbot prompts.

![x](/static/2026-01-07-cp-infinity/08.png)

<br>

Once enabled, we can pretty much use it right away, here are several prompts that we use just to showcase its abilities

![x](/static/2026-01-07-cp-infinity/10.png)

![x](/static/2026-01-07-cp-infinity/11.png)

![x](/static/2026-01-07-cp-infinity/13.png)

![x](/static/2026-01-07-cp-infinity/12.png)

<br>

## Infinity Events

Infinity Events is a centralized, cloud-based security event management solution that unifies logs and security events from all Check Point products (network, endpoint, mobile, IoT, and cloud) into a single pane of glass for monitoring, search, investigation, and threat hunting

![x](/static/2026-01-07-cp-infinity/14.png)

<br>

Here we can see that our Infinity Events is active and getting logs from our On-prem devices, on this dashboard we have some Prevented Attacks that we can see more detailed

![x](/static/2026-01-07-cp-infinity/15.png)

![x](/static/2026-01-07-cp-infinity/16.png)

<br>

It's pretty much a log viewer like on Smartconsole, but on the cloud so its a bit easier to access and manage

![x](/static/2026-01-07-cp-infinity/17.png)

<br>

It has some AI capabilities that's pretty useful, for example here we query the logs but only by prompting it

![x](/static/2026-01-07-cp-infinity/18.png)

<br>

It can also generate some reports which may come handy

![x](/static/2026-01-07-cp-infinity/19.png)

![x](/static/2026-01-07-cp-infinity/20.png)

<br>

## Infinity XDR/XPR

Infinity XDR/XPR is Check Point’s unified detection, response, and prevention platform that correlates telemetry across network, endpoint, cloud, and email to identify and stop multi-vector attacks. It provides centralized incident visibility and automated response using ThreatCloud AI.

![x](/static/2026-01-07-cp-infinity/21.png)

<br>

On the XDR/XPR Integration page, here we integrate the soultion that we'd like to use, we can integrarte it with pretty much any brand but for simplicity we will only work with our On-prem Check Point gateways

![x](/static/2026-01-07-cp-infinity/22.png)

<br>

On IOC Management, here we can enable IOC Feeds. These are sources of Indicators of Compromise (e.g. malicious IPs, domains, URLs, file hashes) ingested into the platform for correlation and detection.

![x](/static/2026-01-07-cp-infinity/23.png)

<br>

After doing some malicious testing so simulate attacks, we get an incident hit

![x](/static/2026-01-07-cp-infinity/24.png)

<br>

We can drill down the incident details with some helpful context provided by Check Point's AI

![x](/static/2026-01-07-cp-infinity/25.png)

<br>

This is also supposed to give us detailed timeline for the attacks, but so far i've tried doing various attacks but not many are detected as incidents. Room for improvement

![x](/static/2026-01-07-cp-infinity/26.png)

<br>

## Infinity Playblocks

Infinity Playblocks is Check Point’s automation and orchestration capability that executes predefined response playbooks across network, endpoint, and cloud when a threat is detected.

![x](/static/2026-01-07-cp-infinity/27.png)

<br>

On Connectors, we enable connection to our specified Gateways so Infinity Playblocks can make changes to them

![x](/static/2026-01-07-cp-infinity/28.png)

<br>

Here they have some predefined Automation that we can use, lets enable them all

![x](/static/2026-01-07-cp-infinity/29.png)

<br>

When connected to gateways, playblocks pushes new policy layer that it will use to enforce the rules

![x](/static/2026-01-07-cp-infinity/32.png)

<br>

We will try to tigger the "Common scanner identified by IPS" using simple NMAP scans

![x](/static/2026-01-07-cp-infinity/30.png)

<br>

Here's the breakdown of what the automation actually does, whenever it detects the scanning taking place it will add the source IP into blocked list

![x](/static/2026-01-07-cp-infinity/31.png)

<br>

We use linux to run an NMAP scan to trigger the automation

![x](/static/2026-01-07-cp-infinity/32a.png)

<br>

After a couple minutes, the playblocks has detected the activity, it also shows the originating ip of 172.17.0.5 which is our Kali Linux machine

![x](/static/2026-01-07-cp-infinity/33.png)

![x](/static/2026-01-07-cp-infinity/34.png)

<br>

Playblocks automatically added the IP into blocked sources list

![x](/static/2026-01-07-cp-infinity/35.png)

<br>

Other than the predefined automation, we can also generate new ones using simple prompts. Here we will block the source IP that tries to access a specific Public IP Address

![x](/static/2026-01-07-cp-infinity/36.png)

![x](/static/2026-01-07-cp-infinity/37.png)

<br>

After it's created, enable the automation

![x](/static/2026-01-07-cp-infinity/38.png)

<br>

Next when playblocks detect the matching logs, it triggers the automation and add the source IP into the blocked source as well

![x](/static/2026-01-07-cp-infinity/39.png)

![x](/static/2026-01-07-cp-infinity/40.png)

<br>

These blocked IPs are pushed into the Gateways into the predefined object that's continuously getting update from playblocks

![x](/static/2026-01-07-cp-infinity/41.png)

<br>

We can open the exact json feed used in that object to see the detailed list

![x](/static/2026-01-07-cp-infinity/42.png)

<br>

And when the feed is updated, gateways will immediately drops all traffic for the specified IP Addresses

![x](/static/2026-01-07-cp-infinity/43.png)

<br>


















