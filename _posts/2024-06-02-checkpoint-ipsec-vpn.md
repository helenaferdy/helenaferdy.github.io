---
title: Check Point IPSec VPN with Cisco Router
date: 2024-06-02 17:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point, VPN]
---

A site-to-site IPsec VPN between a Check Point firewall and a Cisco router involves establishing a secure, encrypted connection over the internet to link two separate networks. This configuration ensures that data transmitted between the networks is protected and authenticated, through the use of IPsec protocols and appropriate configuration on both the Check Point device and the Cisco router.

<br>

## Topology

Here's the topology for this lab deployment

![x](/static/2024-06-02-checkpoint-ipsec-vpn/00.png)

<br>

## Configuring IPSec VPN on Check Point

First, create a new network object for the Check Point's LAN Network

![x](/static/2024-06-02-checkpoint-ipsec-vpn/01a.png)

<br>

And also another one for the Cisco Router's LAN Network

![x](/static/2024-06-02-checkpoint-ipsec-vpn/01b.png)

<br>

On Check Point SmartConsole, on Access Control enable IPSec VPN

![x](/static/2024-06-02-checkpoint-ipsec-vpn/01.png)

<br>

Then on VPN Domain, select the LAN Network

![x](/static/2024-06-02-checkpoint-ipsec-vpn/04a.png)

<br>

Next on Link Selection, select the External Interface IP Address, and hit Ok

![x](/static/2024-06-02-checkpoint-ipsec-vpn/04b.png)

<br>

After that, add the Cisco Router as an Externally Managed VPN Gateway

![x](/static/2024-06-02-checkpoint-ipsec-vpn/02.png)

<br>

Enter the name and the IP Address, then select both Firewall and IPSec VPN

![x](/static/2024-06-02-checkpoint-ipsec-vpn/03.png)

<br>

Then on VPN Domain, select the Cisco's LAN Network, and hit Ok

![x](/static/2024-06-02-checkpoint-ipsec-vpn/04.png)

<br>

## Configuring VPN Communities

On Security Policies >> VPN Communities, add a new Meshed VPN with Check Point and Cisco Router as participant gateways

![x](/static/2024-06-02-checkpoint-ipsec-vpn/05.png)

<br>

Next on Encryption, configure the Phase 1 and Phase 2 IKE encryption and hash algorithm

![x](/static/2024-06-02-checkpoint-ipsec-vpn/06.png)

<br>

Then on Shared Secret, configure the shared secret

![x](/static/2024-06-02-checkpoint-ipsec-vpn/07.png)

<br>

Here's how the VPN Community ends up like

![x](/static/2024-06-02-checkpoint-ipsec-vpn/08.png)

<br>

### Configuring Policy

Next on Policy, create a policy to allow traffic going back and forth between these 2 lan network through VPN

![x](/static/2024-06-02-checkpoint-ipsec-vpn/09.png)

<br>

That should do it on the Check Point side.

<br>
<br>

## Configuring IPSec VPN on Cisco Router

First we'll configure an Internet Security Association and Key Management Protocol (ISAKMP) policy on the Cisco Router, this policy defines how the router will establish tunnels for VPNs  using the IPsec protocol suite.

![x](/static/2024-06-02-checkpoint-ipsec-vpn/10.png)

> * crypto isakmp policy 1 : This command starts the configuration of an ISAKMP policy with the priority (or policy number) of 1. Policies are evaluated in order of their priority number, starting with the lowest. A priority of 1 means this policy will be considered first.
> * encryption 3des : This specifies the encryption algorithm to use for securing the VPN tunnel. 3des stands for Triple Data Encryption Standard, which is an enhancement of the original DES algorithm that applies encryption three times to each data block, making it more secure.
> * hash md5 : This sets the hash algorithm used for ensuring data integrity. md5 (Message Digest Algorithm 5) produces a 128-bit hash value, which is used to verify that the data has not been altered during transmission.
> * authentication pre-share : This specifies the method of authentication for the VPN peers. pre-share means pre-shared keys (PSK) are used.
> * group 2 : This determines the Diffie-Hellman group used for key exchange. Group 2 corresponds to a 1024-bit key length, which determines the strength of the cryptographic key exchange process.
> * lifetime 86400 : This sets the lifetime of the ISAKMP security association in seconds. 86400 seconds equals 24 hours. After this period, the ISAKMP security association will need to be re-established.

<br>

Next we configure a pre-shared key for ISAKMP. This pre-shared key will be used for authenticating a VPN peer with a the address of Check Point Firewall

![x](/static/2024-06-02-checkpoint-ipsec-vpn/11.png)

<br>

After that, we define an extended IP access control list (ACL) that will be used to specify which traffic should be protected by IPsec when establishing a VPN tunnel.

![x](/static/2024-06-02-checkpoint-ipsec-vpn/12.png)

<br>

Then we create an IPsec transform set that defines the combination of encryption and authentication protocols that will be used to protect the data being transmitted through the VPN tunnel. 

![x](/static/2024-06-02-checkpoint-ipsec-vpn/13.png)

<br>

Next we configure a crypto map to establish an IPsec VPN connection

![x](/static/2024-06-02-checkpoint-ipsec-vpn/14.png)

> * crypto map c-map 10 ipsec-isakmp : This command creates a crypto map entry named c-map with a sequence number of 10 and specifies that it will use ISAKMP for IPsec. The sequence number 10 determines the order in which the crypto map entries are evaluated.
> * set peer 172.16.1.3 : This specifies the IP address of the remote VPN peer.
> * set transform-set cpsg : This sets the transform set to be used with this crypto map. The transform set named cpsg defines the encryption and authentication protocols to be used (in this case, esp-3des for encryption and esp-md5-hmac for authentication).
> * match address ipsec-acl : This command specifies the access control list (ACL) named ipsec-acl that defines the traffic to be encrypted and sent through the IPsec tunnel.

<br>

Lastly, we apply the previously defined crypto map to a the Gig2 interface on the Cisco router

![x](/static/2024-06-02-checkpoint-ipsec-vpn/15.png)

### Validating VPN Session

Runing "show crypto session detail" shows that the IPSec VPN between this Cisco Router to the Check Point Firewall is established

![x](/static/2024-06-02-checkpoint-ipsec-vpn/16.png)

<br>
<br>


## Testing the IPSec VPN

Now on the PC from the Cisco Router's LAN network, we are able to connect to the pc on the Checkpoint's LAN network

![x](/static/2024-06-02-checkpoint-ipsec-vpn/17.png)

<br>

And back on the Check Point SmartConsole, we can see all the traffic crossing the IPSec VPN

![x](/static/2024-06-02-checkpoint-ipsec-vpn/18.png)

<br>









































