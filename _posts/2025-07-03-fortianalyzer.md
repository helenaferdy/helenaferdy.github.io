---
title: FortiAnalyzer
date: 2025-07-03 07:00:00 +0700
categories: [Security, Fortinet]
tags: [Fortinet]
---

FortiAnalyzer is a centralized logging, analytics, and reporting solution for Fortinet devices. It collects logs from products like FortiGate, FortiWeb, and other Forti Devices as well as other products, enabling real-time visibility, threat analysis, and compliance reporting across the network.

<br>

## Installing FortiAnalyzer

Deploy the VM like any other forti devices, and upon opening the web ui, we can enter forti account to enable free trial

![x](/static/2025-07-03-fortianalyzer/01.png)

![x](/static/2025-07-03-fortianalyzer/02.png)

<br>

When deploying on lab with trial fortigates VM, we have to enable low encrytion mode so it can communicate to fortigate devices

```
config system global
 set enc-algorithm low
 set oftp-ssl-protocol sslv3
 set ssl-low-encryption enable
end
```

<br>

## Adding Fortinet Devices

On Fortigate, enable the FortiAnalyzer on Fabric Connectors and point it to our FAZ

![x](/static/2025-07-03-fortianalyzer/03.png)

<br>

Then authorize the device

![x](/static/2025-07-03-fortianalyzer/04.png)

<br>

After that the devices will show up as on Device Managed on FAZ

![x](/static/2025-07-03-fortianalyzer/05.png)

<br>

FAZ will now start ingesting logs from managed devices, we can see all the logs in Log View

![x](/static/2025-07-03-fortianalyzer/06.png)

<br>

Or we can see the logs based on the device type

![x](/static/2025-07-03-fortianalyzer/07.png)

![x](/static/2025-07-03-fortianalyzer/08.png)

![x](/static/2025-07-03-fortianalyzer/09.png)

<br>

We can also generate reports using predefined report definitions

![x](/static/2025-07-03-fortianalyzer/10.png)

![x](/static/2025-07-03-fortianalyzer/11.png)

![x](/static/2025-07-03-fortianalyzer/12.png)

<br>

## Adding Other Devices

> FortiAnalyzer can function as a **syslog server**, allowing it to **ingest and store syslog data from non-Fortinet devices** such as firewalls, routers, or servers. This enables centralized log collection and basic analysis, though advanced parsing and reporting features are limited for third-party logs compared to Fortinet-native data.

To enable FAZ as Syslog Server, we need to enable ADOM

![x](/static/2025-07-03-fortianalyzer/13.png)

<br>

This will result multiple ADOMs created, the one we are interested in is Syslog

![x](/static/2025-07-03-fortianalyzer/14.png)

<br>

Now on the device that we'd like to send syslog on, configure it to send syslog to FAZ. Here we have a linux server that also acts as a syslog server for Cisco FTD devices, we will forward the logs from this linux to FAZ

![x](/static/2025-07-03-fortianalyzer/15.png)

<br>

Back on FAZ on Root ADOM, the linux server has shown up as Unauthorized

![x](/static/2025-07-03-fortianalyzer/16.png)

<br>

Authorize the device and add it to Syslog ADOM

![x](/static/2025-07-03-fortianalyzer/17.png)

<br>

Switch to Syslog ADOM, we will see the device in Device Manager

![x](/static/2025-07-03-fortianalyzer/18.png)

<br>

And now the syslog from the device can be seen on 'Log View'

![x](/static/2025-07-03-fortianalyzer/19.png)

<br>




















