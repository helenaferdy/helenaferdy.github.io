---
title: Cisco ISE Profiling
date: 2024-11-06 11:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE]
---

Cisco ISE (Identity Services Engine) profiling policies help classify network endpoints by analyzing various attributes like MAC address, DHCP requests, and other network behaviors. These policies allow administrators to apply security and access controls based on endpoint identity.
Custom profiling policy help to define profiling conditions using attributes like MAC address, OUI (Organizationally Unique Identifier), or DHCP fingerprinting. Once configured, Cisco ISE will automatically classify endpoints that match the criteria, categorizing endpoints to its tailored endpoint groups.

<br>

Here we have a working lab running dot1x & mab to give endpoint access to network, but as we can see here we have a bunch of unknown mac address trying to connect to network

![x](/static/2025-02-18-ise-profiling/01.png)

<br>

On the Cisco ISE Live Logs, we see these mac addresses are not hitting any policy rule because its not classified in any endpoint group

![x](/static/2025-02-18-ise-profiling/02.png)

<br>

To automate the process of indetifing these endpoints, we'll create a custom profiling policy on Work Center >> Profiler >> Profiling Policies named "helenacom-devices", here we set this profile to match endpont that has a mac address starting with 00:50:79:66 and automatically put them in an Endpoint Identity Group

![x](/static/2025-02-18-ise-profiling/03.png)

<br>

Then we can use this Endpoint Identity Group as a condition in policy set

![x](/static/2025-02-18-ise-profiling/04.png)

<br>

Now if any of those devices connect to network, ISE will automatically profile it to the configured group

![x](/static/2025-02-18-ise-profiling/05.png)

<br>

And it'll also hit the policy set that we configured, thus giving that endpoint access to the network

![x](/static/2025-02-18-ise-profiling/06.png)

<br>

Additionally, other than using Endpoint Identity Group, we can also set up a Logical Profile based on the Profiling Policy created earlier. This helps to put bunch of Profiling Policies into a single Logical Profile that we can use as condition in Policy Set

![x](/static/2025-02-18-ise-profiling/06.png)

<br>

And here we put that Logical Profile to use as a condition

![x](/static/2025-02-18-ise-profiling/07.png)

<br>








