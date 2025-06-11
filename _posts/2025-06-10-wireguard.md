---
title: Wireguard VPN
date: 2025-06-10 07:30:00 +0700
categories: [Other, Linux]
tags: []
---


WireGuard is a fast, modern VPN that creates secure point-to-point tunnels using simple configuration and strong cryptography. When set up on a Linux VPS, it can securely tunnel traffic from a client device through the VPS, masking the client's IP and routing its internet activity via the VPS.

<br>

## Installation

Download and run the installer from this [Github Repository](https://github.com/angristan/wireguard-install)

```
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
./wireguard-install.sh
```

![x](/static/2025-06-10-wireguard/01.png)

<br>

On installation, we pretty much just need to click enter for the VPN configuration because the installer will provide default value for each parameter

![x](/static/2025-06-10-wireguard/02.png)

<br>

Same goes with the Client Configuration

![x](/static/2025-06-10-wireguard/04.png)

<br>

After that, we should get the configuration file that we can scan or get directly on the root directory. <br>
Notice there's an error on this installation, thats because my VPS doesn't support IPV6

![x](/static/2025-06-10-wireguard/05.png)

<br>

To overcome the issue, open the wireguard config and delete any IPV6 references

```
nano /etc/wireguard/wg0.conf
```

![x](/static/2025-06-10-wireguard/06.png)

<br>

Then restart the service, and we should be good to go

```
systemctl restart wg-quick@wg0
systemctl status wg-quick@wg0
```

![x](/static/2025-06-10-wireguard/07.png)

<br>

Now that we solved that, we can now copy the configuration file over to our client

![x](/static/2025-06-10-wireguard/08.png)

<br>

Install WireGuard Client and import the config

![x](/static/2025-06-10-wireguard/09.png)

<br>

Activate it, and we are connected

![x](/static/2025-06-10-wireguard/10.png)

<br>

Now our traffic is tunneled to go out of the VPS' public interface

![x](/static/2025-06-10-wireguard/12.png)

<br>


On the Server, run "wg show" to see VPN connections

![x](/static/2025-06-10-wireguard/11.png)

<br>






