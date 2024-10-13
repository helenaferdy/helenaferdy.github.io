---
title: Check Point VSX
date: 2024-09-04 09:00:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point VSX (Virtual System Extension) is a virtualization technology that is designed to create multiple virtual systems (VSs) on a single physical appliance. Each VS operates as an independent firewall with its own policies, routing tables, interfaces, and network resources.

![x](/static/2024-09-04-checkpoint-vsx/00.png)

<br>

## Enabling VSX

On the VSX Gateway, run "set vsx on" to enable VSX

![x](/static/2024-09-04-checkpoint-vsx/01.png)

<br>

Then on SmartConsole, select New >> VSX >> Gateway and enter the details

![x](/static/2024-09-04-checkpoint-vsx/02.png)

![x](/static/2024-09-04-checkpoint-vsx/03.png)

![x](/static/2024-09-04-checkpoint-vsx/04.png)

![x](/static/2024-09-04-checkpoint-vsx/05.png)

![x](/static/2024-09-04-checkpoint-vsx/06.png)

<br>

Now the VSX Gateway is up and running

![x](/static/2024-09-04-checkpoint-vsx/07.png)

<br>

## Creating Virtual System

To add a VS, select New >> VSX >> Virtual System, give it a name and select the VSX Gateway

![x](/static/2024-09-04-checkpoint-vsx/08.png)

<br>

Then configure the interfaces and ip routings

![x](/static/2024-09-04-checkpoint-vsx/09.png)

<br>

After that hit Finish

![x](/static/2024-09-04-checkpoint-vsx/10.png)

<br>

Now we have a Virtual System up and running

![x](/static/2024-09-04-checkpoint-vsx/11.png)

<br>

## Policies

Here we create a new policy and set the VS-1 as the installation target

![x](/static/2024-09-04-checkpoint-vsx/12.png)

<br>

Then we create a simple policy to allow internet access from the downlink nodes

![x](/static/2024-09-04-checkpoint-vsx/13.png)

<br>

Next hit install on the policy

![x](/static/2024-09-04-checkpoint-vsx/14.png)

<br>

On the client side, we now have internet access going through the VS-1 Firewall

![x](/static/2024-09-04-checkpoint-vsx/15.png)

![x](/static/2024-09-04-checkpoint-vsx/16.png)

<br>

## Creating Second Virtual System

We now know the drill, repeat the process to create the 2nd VS

![x](/static/2024-09-04-checkpoint-vsx/17.png)

<br>

For simplicity, we'll use the same policy for this VS-2

![x](/static/2024-09-04-checkpoint-vsx/18.png)

<br>

And we also have internet access now on the VS-2 clients

![x](/static/2024-09-04-checkpoint-vsx/19.png)

![x](/static/2024-09-04-checkpoint-vsx/20.png)

<br>

## VSX Validation

On the VSX Gateway, run "vsx stat -v" to see the VS status

![x](/static/2024-09-04-checkpoint-vsx/21.png)

<br>

Command "vsx stat -l" gives a little bit more details on these VSes

![x](/static/2024-09-04-checkpoint-vsx/22.png)

<br>

And run "vsenv" to switch between the VS context, as shown below each context has its own configuration

![x](/static/2024-09-04-checkpoint-vsx/23.png)

<br>

