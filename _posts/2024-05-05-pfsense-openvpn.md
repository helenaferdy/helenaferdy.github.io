---
title: pfSense OpenVPN Remote Access
date: 2024-05-05 07:30:00 +0700
categories: [Security, pfSense]
tags: [pfSense, OpenVPN, VPN]
---

<br>

OpenVPN remote access allows users to securely connect to a central network over the internet. Users install OpenVPN client software, authenticate with the server, and establish an encrypted tunnel. Once connected, users can access network resources as if they were on-site, with traffic routed securely through the VPN tunnel.

<br>
<br>

Here's the topology of this deployment

![x](/static/2024-05-05-pfsense-openvpn/00.png)

<br>
<br>

## Managing Certificates

> OpenVPN utilizes SSL/TLS protocols for encryption, providing a secure connection over the internet. Certificates play a crucial role in OpenVPN's security framework, as they are used for authentication between the client and the server.

Here we already have a CA server, so lets just add it as our Trusted CA on pfSense on System >> Certificate >> Authorities

![x](/static/2024-05-05-pfsense-openvpn/01.png)

![x](/static/2024-05-05-pfsense-openvpn/02.png)

<br>

Next create a CSR for our pfSense's certificate

![x](/static/2024-05-05-pfsense-openvpn/03.png)

<br>

Sign it on Helena-CA

![x](/static/2024-05-05-pfsense-openvpn/04.png)

<br>

Complete the CSR by adding the final certificate on System >> Certificate >> Certificates

![x](/static/2024-05-05-pfsense-openvpn/05.png)

<br>

Bonus and completely optional, use that certificate on our web server on System >> Advanced >> Admin Access so we get a trusted secure HTTPS connection

![x](/static/2024-05-05-pfsense-openvpn/06.png)

<br>
<br>

## Configuring OpenVPN Remote Access

Before configuring OpenVPN, lets install a package to ease the process of exporting OpenVPN Client Configuration later

![x](/static/2024-05-05-pfsense-openvpn/07.png)

<br>

To configure OpenVPN, go to VPN >> OpenVPN >> Wizard, select the type of Local User Access

![x](/static/2024-05-05-pfsense-openvpn/08.png)

<br>

Here select our trusted CA

![x](/static/2024-05-05-pfsense-openvpn/09.png)

<br>

Then select the newly created certificate

![x](/static/2024-05-05-pfsense-openvpn/10.png)

<br>

Here give it a name, select the interface and used port

![x](/static/2024-05-05-pfsense-openvpn/11.png)

<br>

On Tunnel Settings, configure the Tunnel Network for our VPN Clients to use, then declare the internal network to be accessed by VPN users on IPV4 Local Network. Also disable Redirect IPV4 Gateway to enable Split Tunneling

![x](/static/2024-05-05-pfsense-openvpn/12.png)

> VPN split tunneling is a feature that allows a VPN user to divide their internet traffic into two separate streams: one that is routed through the VPN tunnel to the VPN server, and another that is sent directly to the internet without passing through the VPN.

<br>

Select both firewall rules to be automatically created by the wizard

![x](/static/2024-05-05-pfsense-openvpn/13.png)

<br>

Now the OpenVPN is up and running

![x](/static/2024-05-05-pfsense-openvpn/14.png)

<br>

If we edit the VPN, we can see we have multiple server modes, this setting basically dictates how the authentication is done between clients and server.
Here we select SSL + User Auth meaning clients will be asked to have both personal user certificate and credentials to connect to the VPN.

![x](/static/2024-05-05-pfsense-openvpn/15.png)

<br>
<br>

## Configuring VPN Users

On System >> User Manager, create new user with having user certificate signed by the CA

![x](/static/2024-05-05-pfsense-openvpn/16.png)

<br>
<br>

## Exporting VPN Configurations

On VPN >> OpenVPN >> Client Export, download the Inline Configurations and the VPN Client Installers

![x](/static/2024-05-05-pfsense-openvpn/17.png)

<br>
<br>

## Connecting to VPN

On the Client PC, install the VPN Client

![x](/static/2024-05-05-pfsense-openvpn/18.png)

<br>

Now connect using the exported config and user/password

![x](/static/2024-05-05-pfsense-openvpn/19.png)

<br>

And we have successfully connected to the VPN

![x](/static/2024-05-05-pfsense-openvpn/20.png)

<br>

Back on pfSense, on Status >> OpenVPN, we can see the user helena is connected

![x](/static/2024-05-05-pfsense-openvpn/21.png)

<br>

On Linux, we can simply run the command "openvpn" followed by the exported config, then enter credentials

![x](/static/2024-05-05-pfsense-openvpn/22.png)

<br>

We can see that the linux client has also been connected

![x](/static/2024-05-05-pfsense-openvpn/23.png)

<br>

