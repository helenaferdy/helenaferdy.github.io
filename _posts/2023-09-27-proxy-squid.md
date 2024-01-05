---
title: Squid Proxy on Ubuntu
date: 2023-09-27 07:30:00 +0700
categories: [Proxy, Squid]
tags: [Proxy]
---

<br>

Squid is a free and open-source caching proxy server for HTTP, HTTPS, FTP, and other popular network protocols. It acts as an intermediary between web servers and clients, caching frequently requested content to improve performance and reduce bandwidth usage.

<br>
<br>

## Connection Topology

Here's the connection topology, where all the HTTP/HTTPS traffic from client will go through the proxy server first before going to the firewall then the internet

![x](/static/2023-09-27-proxy-squid/00.png)

<br>
<br>

## Installing Squid Proxy on Ubuntu

Run this command to install Squid Proxy on Ubuntu

```shell
sudo apt install squid
```

<br>

After installed, run this command to see the service status

```shell
sudo systemctl status squid
```

![x](/static/2023-09-27-proxy-squid/01.png)

<br>
<br>

## Configuring Squid Proxy

Now let's configure the proxy. First rename the original config file so we can start with a clean sheet

```shell
sudo mv /etc/squid/squid.conf /etc/squid/squid.conf.original
```

<br>

Then create a new one with this command

```shell
sudo nano /etc/squid/squid.conf
```

<br>

This simple configuration starts the proxy on 8080 port and allows all traffic from the client on 62.0.0.0/24 segment

```shell
http_port 8080
acl localnet src 62.0.0.0/24
http_access allow localnet
http_access allow localhost
```

<br>

Run this command to validate the configuration

```shell
sudo squid -k parse
```

![x](/static/2023-09-27-proxy-squid/01a.png)

<br>

Then restart the service for the configuration to take effect

```shell
sudo systemctl restart squid
```

<br>

Now let's try connecting to internet using proxy with this command

```shell
curl -v -x http://62.0.0.10:8080 https://www.x.com
```

<br>

We can see the connection to proxy is working and we're able to access the internet

![x](/static/2023-09-27-proxy-squid/02.png)

<br>

Run this command to see the traffic logs

```shell
sudo tail -n 100 /var/log/squid/access.log
```

![x](/static/2023-09-27-proxy-squid/03.png)

<br>
<br>

## Firewall Configuration

Next is to allow web access only if the clients use the proxy, to do that we create two rules on the firewall

![x](/static/2023-09-27-proxy-squid/04.png)

The first rule allows web access only for traffic originating from the proxy server, and the second one blocks all web traffic from other clients

<br>
<br>

## Testing from Client PC

On the client PC, configure it to use the proxy

![x](/static/2023-09-27-proxy-squid/05.png)

<br>

Now only the connection using proxy will be able to access internet

![x](/static/2023-09-27-proxy-squid/06.png)

> chrome (left) doesnt have proxy configured, while firefox (right) does

<br>

Checking the traffic logs on the firewall, we should see only traffic from the proxy server is allowed

![x](/static/2023-09-27-proxy-squid/07.png)

<br>

Traffic logs can also be seen on the squid proxy server

![x](/static/2023-09-27-proxy-squid/08.png)

<br>
<br>


## Proxy Server Site Blocking

Now lets try blocking traffic to facebook and reddit, on the sqiuid.conf file add these lines

```shell
acl block_sites dstdomain .facebook.com .reddit.com
http_access deny block_sites
```

![x](/static/2023-09-27-proxy-squid/11a.png)

<br>

Now if we access facebook or reddit, we should see error saying proxy refuses the connection

![x](/static/2023-09-27-proxy-squid/09.png)

<br>

On the squid logs we can also see the denied traffic

![x](/static/2023-09-27-proxy-squid/10.png)

<br>
