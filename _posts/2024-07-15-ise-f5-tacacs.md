---
title: Cisco ISE TACACS+ with F5 Virtual Server
date: 2024-07-15 10:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE, TACACS, F5]
---

This configuration examines the use of an F5 Virtual Server as a load balancer for Cisco ISE TACACS+ servers. By configuring the F5 to balance TACACS+ traffic, it optimizes resource utilization and enhances fault tolerance across Cisco ISE instances, ensuring reliable and efficient management of network device access and security policies.

![x](/static/2024-07-15-ise-f5-tacacs/00.png)

<br>

## Configuring F5 Virtual Server

First lets configure a new Health Monitor to monitor the ISE's tacacs service on port 49, on Local Traffic >> Monitors create a new one

![x](/static/2024-07-15-ise-f5-tacacs/01.png)

<br>

Next create a Pool containing the 2 ISE Servers with service port on port 49 and using the Health Monitor created

![x](/static/2024-07-15-ise-f5-tacacs/02.png)

![x](/static/2024-07-15-ise-f5-tacacs/03.png)

<br>

Then we create a new Virtual Server pointing to the Virtual IP with member being the ISE Pool

![x](/static/2024-07-15-ise-f5-tacacs/04.png)

![x](/static/2024-07-15-ise-f5-tacacs/05.png)

![x](/static/2024-07-15-ise-f5-tacacs/06.png)

<br>

## Testing Virtual Server

Now the Virtual IP should be accessible on port 49

![x](/static/2024-07-15-ise-f5-tacacs/07.png)

<br>

Configure the tacacs on the NAD to point to the Virtual IP

```shell
aaa new-model

tacacs server ISE_VS
 address ipv4 172.16.1.100
 key helena

aaa group server tacacs+ TACACS_SVR
 server name ISE_VS

aaa authentication login default group TACACS_SVR local
aaa authentication enable default group TACACS_SVR enable 
aaa accounting update newinfo
aaa accounting exec default start-stop group TACACS_SVR
aaa accounting commands 0 default start-stop group TACACS_SVR
aaa accounting commands 1 default start-stop group TACACS_SVR
aaa accounting commands 15 default start-stop group TACACS_SVR
aaa authorization config-commands
aaa authorization exec default group TACACS_SVR local if-authenticated
aaa authorization commands 0 default group TACACS_SVR local if-authenticated
aaa authorization commands 1 default group TACACS_SVR local if-authenticated 
aaa authorization commands 15 default group TACACS_SVR local if-authenticated 
```

<br>

On both ISE Servers, because the default gateway is not pointing to the F5, we need to engineer the traffic coming from NAD though F5 to be returned back to F5 with static routes

![x](/static/2024-07-15-ise-f5-tacacs/08.png)

<br>

Now if we try to authenticate to the NAD, F5 will load balance the request to both ISE Servers

![x](/static/2024-07-15-ise-f5-tacacs/09.png)

<br>

The status of both ISE server can also be monitored on F5

![x](/static/2024-07-15-ise-f5-tacacs/11.png)

<br>

## Shutting Down ISE Service

If we shutdown the ISE service, this will make the First Node to be inaccessible on port 49 but is still ping-able

![x](/static/2024-07-15-ise-f5-tacacs/12.png)

<br>

On F5 monitor we can see that the node is detected down

![x](/static/2024-07-15-ise-f5-tacacs/13.png)

<br>

And for that, F5 will forward the traffic to only the second node

![x](/static/2024-07-15-ise-f5-tacacs/14.png)

<br>































