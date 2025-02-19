---
title: Cisco ISE Posture
date: 2024-11-05 11:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE]
---


Cisco ISE posture assessment is a feature that checks the compliance status of endpoint devices connecting to the network. It ensures that these devices meet security requirements—such as having up-to-date antivirus, patches, or other mandated software—before granting them access.

There are 2 scenarios that we cover on this deployment :
> * Temporal Agent : It is a lightweight, temporary application downloaded each time a user logs in, used only for that session, and removed after disconnecting. This option is typically for guest users or temporary devices.
> * AnyConnect Agent : It is a persistent agent installed on the endpoint device, ideal for corporate-managed devices. It provides continuous posture assessment, allowing ISE to enforce security policies consistently on devices connecting to the network.

<br>

Here's the flow of Posture Configuration

![x](/static/2024-11-05-ise-posture/01.png)

> * Posture Conditions : Define the specific conditions or checks that endpoints must meet (e.g., antivirus status, OS patch level).
> * Posture Remediations : Actions that endpoints need to take if they fail to meet the posture conditions (e.g., update antivirus, install patches).
> * Posture Requirements : Combine multiple posture conditions and remediations, setting compliance criteria for access.
> * Posture Policy : Policies applied to endpoints that determine if they are compliant or need remediation based on posture requirements.
> * Client Provisioning : Provisioning of the necessary agent (e.g., Temporal or AnyConnect) on the endpoint for posture assessment.
> * Access Policy : Final decision on access level, granting or limiting network access based on the endpoint's compliance status.

<br>

## Temporal Agent

### Posture Condition

First we configure the Posture Condition on Work Centers >> Policy Elements >> Conditions, here we make one conditon example where windows endpoints need to have Firewall Enabled

![x](/static/2024-11-05-ise-posture/02.png)

<br>

### Posture Remediation

Next the Remediation, this opstion don't really work when using Temporal Agent but let's configure it anyway. This Remediation Policy will automatically try to remediate when Firewall is not enabled by enabling them on endpoints

![x](/static/2024-11-05-ise-posture/03.png)

<br>

### Posture Requirements

Next we put together the Condition and Remediation into a Posture Requirements, where here we configure the compliance criteria. For the remediaton we can only show a Message Text because of using the Temporal Agent

![x](/static/2024-11-05-ise-posture/04.png)

<br>

### Posture Policy

Next we create a Posture Policy that points to the Posture Requirement we just made. This policy determines whether the enpdoint is compliant, requiring remediation, or not compliant

![x](/static/2024-11-05-ise-posture/05.png)

<br>

### Client Provisioning

After that we configure the Client Provisioning Policy where we set what software to use for posturing. Here we create a specific rule to provision Windows 10 Endpoints to use Temporal Agent

![x](/static/2024-11-05-ise-posture/06.png)

<br>

### Access Policy

Next we create a Authz Profile with ACL to only access ISE and a Web Redirection redirecting to the Default Client Provisioning Portal

![x](/static/2024-11-05-ise-posture/07.png)

> the "ACL_REDIRECT" ACL must exist on the NAD

<br>

Then we create Policy Sets that govern the use cases whether the endpoint is Not Yet Postured (Unknown), Compliant, or Not Compliant

![x](/static/2024-11-05-ise-posture/08.png)

<br>

### Accessing the Network

Now if an endpoint accesses the network, it will get the Posturing Policy that redirects it to the Client Provisioning Portal

![x](/static/2024-11-05-ise-posture/09.png)

<br>

On the Endpoint side, we're redirected to the Posturing Portal

![x](/static/2024-11-05-ise-posture/10.png)

<br>

Because we don't have any agnet installed, we're asked to download and install the Temporal Agent

![x](/static/2024-11-05-ise-posture/11.png)

![x](/static/2024-11-05-ise-posture/12.png)

<br>

And after the check is done, we will be given the final verdict of compliant and a network access is granted

![x](/static/2024-11-05-ise-posture/13.png)

<br>

Here how the process looks like on the Live Logs

![x](/static/2024-11-05-ise-posture/14.png)

<br>

### Non Compliant Endpoint

Now lets try accessing the network while having Firewall Disabled 

![x](/static/2024-11-05-ise-posture/15.png)

<br>

When running the Temporal Agent again, the endpoint is not compliant. Its also showing the text message remediation

![x](/static/2024-11-05-ise-posture/16.png)

![x](/static/2024-11-05-ise-posture/17.png)

<br>

## AnyConnect Agent

### Posture Requirements

Here we create a new Posture Requirement using the same Condition but with Posture Type of Agent (AnyConnect) and Remediation of Win_FW_Remed where it will try to automatically enable the Firewall if its disabled

![x](/static/2024-11-05-ise-posture/20.png)

<br>

### Posture Policy

We're also adding a new Posture Policy to use the new Posture Requirement

![x](/static/2024-11-05-ise-posture/21.png)

<br>

### Client Provisioning

For Agent Client Provisioning, we need to import the AnyConnect compliance package and AnyConnect Desktop Program into Client Provisioing Resources

![x](/static/2024-11-05-ise-posture/22.png)

<br>

Next we also need to create a Agent Profile to use the both packages

![x](/static/2024-11-05-ise-posture/23.png)

![x](/static/2024-11-05-ise-posture/24.png)

<br>

Finally on Client Provisioning Policy, we modify the policy so it uses the Agent Profile just created

![x](/static/2024-11-05-ise-posture/25.png)

<br>

### Access Policy

Next on the Authz Profile, here we need to enable Internet Access so endpoint can download the AnyConnect package if its not installed yet, and a Web Redirection redirecting to Default Client Provisioing Portal

![x](/static/2024-11-05-ise-posture/26.png)

<br>

Next the same drill where we configure the Policy Sets

![x](/static/2024-11-05-ise-posture/27.png)

<br>

### Accessing the Network

Connecting endpoint to the network will give it a Posturing Policy that redirects it to the Client Provisioning Portal

![x](/static/2024-11-05-ise-posture/28.png)

<br>

On the Endpoint side, we will be asked to download and install the AnyConnect Agent if its not yet installed

![x](/static/2024-11-05-ise-posture/29.png)

![x](/static/2024-11-05-ise-posture/30.png)

<br>

And after the AnyConnect agent assesses the endpoint, it verdicts that the endpoint is compliant

![x](/static/2024-11-05-ise-posture/31.png)

![x](/static/2024-11-05-ise-posture/32.png)

<br>

### Non Compliant Endpoint

Now we try accessing the network with Firewall Disabled

![x](/static/2024-11-05-ise-posture/33.png)

<br>

Because the AnyConnect agent has a remediation capability, it automaticaly enables the Firewall so the endpoint becomes compliant

![x](/static/2024-11-05-ise-posture/34.png)

<br>
<br>
<br>
<br>

Oh yea, this is the configuration on the cisco switch side

```text
radius server ISE1
 address ipv4 198.18.128.11 auth-port 1812 acct-port 1813
 key helena

radius server ISE2
 address ipv4 198.18.128.12 auth-port 1812 acct-port 1813
 key helena


aaa group server radius ISE
 server name ISE1
 server name ISE2

aaa authentication dot1x default group ISE
aaa authorization network default group ISE
aaa accounting dot1x default start-stop group ISE
aaa accounting system default start-stop group ISE
aaa accounting update periodic 5
aaa accounting update newinfo periodic 2880
!
radius-server dead-criteria time 5 tries 3
radius-server deadtime 15
radius-server retry method reorder
radius-server transaction max-tries 3
!
aaa server radius dynamic-author
 client 198.18.128.11 server-key helena
 client 198.18.128.12 server-key helena
!
device-sensor filter-list dhcp list DHCP-LIST
 option name host-name
 option name requested-address
 option name parameter-request-list
 option name class-identifier
 option name client-identifier
 option name user-class-id
 device-sensor filter-spec dhcp include list DHCP-LIST
!
cdp run
!
device-sensor filter-list cdp list CDP-LIST
 tlv name device-name
 tlv name address-type
 tlv name capabilities-type
 tlv name version-type
 tlv name platform-type
!
device-sensor filter-spec cdp include list CDP-LIST
!
lldp run
!
device-sensor filter-list lldp list LLDP-LIST
 tlv name system-name
 tlv name system-description
 tlv name system-capabilities
device-sensor filter-spec dhcp include list DHCP-LIST
device-sensor filter-spec lldp include list LLDP-LIST
device-sensor filter-spec cdp include list CDP-LIST
device-sensor accounting
device-sensor notify all-changes
!
device-sensor filter-spec lldp include list LLDP-LIST
!
radius-server vsa send accounting
radius-server vsa send authentication
radius-server attribute 6 on-for-login-auth
radius-server attribute 6 support-multiple
radius-server attribute 8 include-in-access-req
radius-server attribute 25 access-request include
radius-server attribute 31 mac format ietf upper-case
radius-server attribute 31 send nas-port-detail mac-only
!
ip device tracking probe auto-source
ip device tracking probe delay 10			
!
device-tracking tracking				
device-tracking policy IPDT_POLICY			
 no protocol udp					
 tracking enable					
!
dot1x critical eapol
!
ip http server
ip http active-session-modules none
ip http secure-active-session-modules none
ip domain-name belajar.local
crypto key generate rsa general-keys mod 2048
ip http secure-server
ip http max-connections 48
!
dot1x system-auth-control
!
authentication mac-move permit
!

interface g1/1
 switchport access vlan 10
 switchport mode access
 authentication host-mode multi-domain
 authentication order dot1x mab
 authentication priority dot1x mab 
 authentication port-control auto
 mab
 dot1x pae authenticator
```








