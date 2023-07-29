---
title: Configure Firewall Policy on Fortigate
date: 2023-07-26 20:30:00 +0700
categories: [Security, Fortigate]
tags: [fortigate]
---

<br>

Right now i have one linux server that cannot access anything on the internet

![01](/static/2023-07-26-fortigate-firewall/01.png)

<br>

That is because on the Fortigate firewall, i only have one policy which denies all access to internet

![02](/static/2023-07-26-fortigate-firewall/02.png)

<br>

<br>


## Allowing traffic to Telegram API

<br>

This linux server has one cronjob that sends a telegram message every one minute, so the plan is to set up a firewall policy to give this host an access to only the telegram API so it can do its job.

<br>

First, open Fortigate and got to Policy & Objects -> Firewall Policy, and create a new Policy

![03](/static/2023-07-26-fortigate-firewall/03.png)

I named this policy "allow telegram", with the source host of the linux server and the destination of telegram API on api.telegram.org with only allowing HTTPS traffic.
I also set up NAT because this traffic is going to the internet.

<br>

![04](/static/2023-07-26-fortigate-firewall/04.png)

For the security profiles, i left it as it is as it is not really important for this scenario.
Then i toggled on the Log Allowed Traffic with All Sessions so i can inspect all the allowed traffic later on.

<br>

<br>


## Allowing traffic to DNS Server

<br>

I created another new policy to allow access to the DNS Server of 1.1.1.1 so the linux server is able to resolve the FQDN of api.telegram.org into an IP address.

![05](/static/2023-07-26-fortigate-firewall/05.png)

The configuration is pretty much the same as the one created before.

<br>

And this is the final Firewall Policy that we have.

![06](/static/2023-07-26-fortigate-firewall/06.png)


<br>

<br>

## Testing the internet access

<br>

Doing a curl to https://api.telegram.org should be able to simulate an api connection to Telegram as well as a DNS resolving traffic to 1.1.1.1

![07](/static/2023-07-26-fortigate-firewall/07.png)

<br>

To monitor the allowed traffic, go to Log & Report -> Forward Traffic, the HTTPS traffic to Telegram API Endpoint and the DNS traffic to 1.1.1.1 should show up, with other additional details such as the timestamp and the corresponding firewall policy that's being hit.

![08](/static/2023-07-26-fortigate-firewall/08.png)

