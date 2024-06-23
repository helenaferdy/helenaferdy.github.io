---
title: Fortinet SD-WAN Configuration with Fortimanager Templates
date: 2023-10-26 01:30:00 +0700
categories: [SD-WAN, Fortinet SD-WAN]
tags: [SD-WAN, Fortigate, VPN]
---

<br>

Fortimanager is a centralized network security management and automation platform for Fortigate Firewalls, and here we use its template push feature as a way to create, manage, and enforce configurations and policies across all managed devices

<br>
<br>

## Network Topology

Here's the network topology for this deployment that will be configured by templates

![x](/static/2023-10-26-fortimanager-template/00.png)

<br>
<br>

## Creating CLI Templates

On Fortimanager, we have 3 managed devices that has been grouped by their roles, where one will become the SD-WAN hub and the other 2 will be the edges

![x](/static/2023-10-26-fortimanager-template/01.png)

<br>

To configure the templates, go to Device Manager >> Provisioning Templates, create CLI Templates for Hub and Edges

### Hub

> 01-Hub-IPSec_VPN

```conf
config vpn ipsec phase1-interface
    edit "$(VPN_1)"
        set type dynamic
        set interface "port1"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal des-sha1
        set add-route disable
        set dpd on-idle
        set auto-discovery-sender enable
        set nattraversal disable
        set psksecret ENC 6aFFWWvRVMemgcv6Ci0r7H2FckqjmT4kDxO6zC5pROZejr84nvKZHOkWB32lvFJ5SjTwXRQ0FYYyotyCFt5kzO18pkZ0a900iiWtfv4Nij0dkO88GnQVsudR5D4bVraY6KD2atorngrfGgR2LEMdQzkQmZ5MaJdA8lyEJUW00aDuHXmj4zEczM33apTMBcixhokzOw==
        set dpd-retryinterval 60
    next
    edit "$(VPN_2)"
        set type dynamic
        set interface "port2"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal des-sha1
        set add-route disable
        set dpd on-idle
        set auto-discovery-sender enable
        set nattraversal disable
        set psksecret ENC ceaaymqdVVxUPeV1l6EK8JgoFFswet7t1rKgqSrw1fsSdl0JYUFfUw9q76W1khLEiyK4gr2191NtLSuMr5GqebVfv5q82yy0r0xTHpk7Njtu9oPbWSwBU8rflVGYV49M6DEG3MX1V5eEWhjTJKNa3snQ5RyvANymvib78UBGEP1cNn45Fl77Lol2UBOjEAA9CjnoHQ==
        set dpd-retryinterval 60
    next
end
config vpn ipsec phase2-interface
    edit "$(VPN_1)"
        set phase1name "$(VPN_1)"
        set proposal des-sha1
    next
    edit "$(VPN_2)"
        set phase1name "$(VPN_2)"
        set proposal des-md5
    next
end   
```

> 02-Hub-SDWAN

```conf
config system sdwan
    set status enable
    set load-balance-mode measured-volume-based
    config zone
        edit "virtual-wan-link"
        next
        edit "HELENA"
        next
    end
    config members
        edit 1
            set interface "$(VPN_1)"
            set zone "HELENA"
            set source $(LOCAL_GATEWAY)
        next
        edit 2
            set interface "$(VPN_2)"
            set zone "HELENA"
            set source $(LOCAL_GATEWAY)
        next
    end
```

> 03-Hub-BGP

```conf
config router bgp
    set as 65001
    set router-id $(BGP_ROUTER_ID)
    set keepalive-timer 5
    set ibgp-multipath enable
    set additional-path enable
    config neighbor-group
        edit "$(VPN_1)"
            set capability-graceful-restart enable
            set soft-reconfiguration enable
            set interface "$(VPN_1)"
            set remote-as 65001
            set route-reflector-client enable
        next
        edit "$(VPN_2)"
            set capability-graceful-restart enable
            set soft-reconfiguration enable
            set interface "$(VPN_2)"
            set remote-as 65001
            set route-reflector-client enable
        next
    end
    config neighbor-range
        edit 1
            set prefix 10.100.100.0 255.255.255.0
            set neighbor-group "$(VPN_1)"
        next
        edit 2
            set prefix 10.200.200.0 255.255.255.0
            set neighbor-group "$(VPN_2)"
        next
    end
    config network
        edit 1
            set prefix $(LOCAL_PREFIX) 255.255.255.0
        next
    end
```

> 04-Hub-VPN_Overlay_Interface

```conf
config system interface
    edit "$(VPN_1)"
        set vdom "root"
        set ip $(BGP_GATEWAY_1) 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 10.100.100.254 255.255.255.0
        set snmp-index 4
        set interface "port1"
    next
    edit "$(VPN_2)"
        set vdom "root"
        set ip $(BGP_GATEWAY_2) 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 10.200.200.254 255.255.255.0
        set snmp-index 5
        set interface "port2"
    next
```

> 05-Hub-Firewall_Policy

```conf
config firewall policy
    edit 1
        set name "edge_to_hub"
        set srcintf "HELENA"
        set dstintf "port3"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
    edit 2
        set name "edge_to_edge"
        set srcintf "HELENA"
        set dstintf "HELENA"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
    edit 3
        set name "hub_to_edge"
        set srcintf "port3"
        set dstintf "HELENA"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

### Edges

> 11-Edge-IPSec_VPN

```conf
config vpn ipsec phase1-interface
    edit "$(VPN_1)"
        set interface "port1"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal des-sha1
        set add-route disable
        set auto-discovery-receiver enable
        set nattraversal disable
        set remote-gw 198.1.0.2
        set psksecret ENC d7BjaavtSimgDzDr5weTdhw75d3AryY051HdpvLFGBPcMWwhorM4kMjedkSk0K1ddv83a7FCTrPFkGjA/QHJgt3vgzQMX/UgL40zpV/kw7uMeRo/8bQmNaT92nz9YNcMz/faVjP0iy+D92cwOIA0xANh/ogB7aeWYpkliJAvsE8mjxb5k7YAAU69+GPaXk7YZ6g5IQ==
    next
    edit "$(VPN_2)"
        set interface "port2"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal des-sha1
        set add-route disable
        set auto-discovery-receiver enable
        set nattraversal disable
        set remote-gw 197.1.0.2
        set psksecret ENC tg9K/aQnNFQJti++ZilCxwrv92RAKqCJK4mHWSFKCe5ZSBkBFedygK8vo3an5QbOdK0DHh5rtnSlE1LrYBNbi6wUkNQxERO9JQGzE544E0TXI/DoIFB0uwNRKTcEqn4xBK8nNY0a7rJp1yLW9tkB7ta5QhswaXa3NT7259CLvtbHZs00erpMb53tVcwtDp5ibpyZuQ==
    next
end
config vpn ipsec phase2-interface
    edit "$(VPN_1)"
        set phase1name "$(VPN_1)"
        set proposal des-sha1
    next
    edit "$(VPN_2)"
        set phase1name "$(VPN_2)"
        set proposal des-md5
    next
end
```

> 12-Edge-SDWAN

```conf
config system sdwan
    set status enable
    set load-balance-mode measured-volume-based
    config zone
        edit "virtual-wan-link"
        next
        edit "HELENA"
        next
    end
    config members
        edit 1
            set interface $(VPN_1)
            set zone "HELENA"
            set source $(LOCAL_GATEWAY)
        next
        edit 2
            set interface $(VPN_2)
            set zone "HELENA"
            set source $(LOCAL_GATEWAY)
        next
    end
```

> 13-Edge-BGP

```conf
config router bgp
    set as 65001
    set router-id $(BGP_ROUTER_ID)
    set keepalive-timer 5
    set ibgp-multipath enable
    set additional-path enable
    config neighbor
       edit 10.100.100.1
	        set advertisement-interval 1
            set capability-graceful-restart enable
            set soft-reconfiguration enable
            set remote-as 65001
       next
       edit 10.200.200.1
	        set advertisement-interval 1
            set capability-graceful-restart enable
            set soft-reconfiguration enable
            set remote-as 65001
        next
    end
    config network
        edit 1
            set prefix $(LOCAL_PREFIX) 255.255.255.0
        next
    end
```

> 14-Edge-VPN_Overlay_Interface

```conf
config system interface
    edit "$(VPN_1)"
        set vdom "root"
        set ip $(BGP_GATEWAY_1) 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 10.100.100.254 255.255.255.0
        set snmp-index 4
        set interface "port1"
    next
    edit "$(VPN_2)"
        set vdom "root"
        set ip $(BGP_GATEWAY_2) 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 10.200.200.254 255.255.255.0
        set snmp-index 5
        set interface "port2"
    next
end
```

> 15-Edge-Firewall_Policy

```conf
config firewall policy
    edit 1
        set name "sdwan_out"
        set srcintf "port3"
        set dstintf "HELENA"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
    edit 2
        set name "sdwan_in"
        set srcintf "HELENA"
        set dstintf "port3"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

<br>

Here's what we end up with

![x](/static/2023-10-26-fortimanager-template/02.png)

<br>

Next, create a Template Group containing the templates created

![x](/static/2023-10-26-fortimanager-template/03.png)

![x](/static/2023-10-26-fortimanager-template/04.png)

<br>

After that, assign the device for the Template Group

![x](/static/2023-10-26-fortimanager-template/05.png)

![x](/static/2023-10-26-fortimanager-template/06.png)

<br>

Do the same for the edges

![x](/static/2023-10-26-fortimanager-template/07.png)

<br>
<br>

## Configuring Metadata Mapping

After that, go to the devices and select Edit Variable Mapping

![x](/static/2023-10-26-fortimanager-template/11.png)

<br>

Configure the metadata for each device

![x](/static/2023-10-26-fortimanager-template/12.png)

![x](/static/2023-10-26-fortimanager-template/13.png)

![x](/static/2023-10-26-fortimanager-template/14.png)

<br>
<br>

## Creating Policy Package

Next, create two Policy Packages for Hub and Edges, this will be used to push the templates to the devices

![x](/static/2023-10-26-fortimanager-template/08.png)

![x](/static/2023-10-26-fortimanager-template/09.png)

<br>

Here's the two policy packages

![x](/static/2023-10-26-fortimanager-template/10.png)

<br>
<br>

## Pushing the Configurations

Now it's time to push the configuration to the devices, select Install Wizard then choose the Hub Policy Package

![x](/static/2023-10-26-fortimanager-template/15.png)

<br>

Make sure the validation is all success, then proceed with installation

![x](/static/2023-10-26-fortimanager-template/16.png)

<br>

After the installation completes, we can see the configurations have been pushed to the actual device

![x](/static/2023-10-26-fortimanager-template/17.png)

<br>

Now do the push for the Edges

![x](/static/2023-10-26-fortimanager-template/18.png)

![x](/static/2023-10-26-fortimanager-template/19.png)

<br>

And it looks like the push is also a success on the edges

![x](/static/2023-10-26-fortimanager-template/20.png)

![x](/static/2023-10-26-fortimanager-template/21.png)

<br>

Going to the Hub, the SD-WAN configuration is up and running

![x](/static/2023-10-26-fortimanager-template/22.png)

<br>

















