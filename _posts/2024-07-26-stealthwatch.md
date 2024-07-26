---
title: Cisco Secure Network Analytics (Stealthwatch)
date: 2024-07-26 07:30:00 +0700
categories: [Security, Cisco Secure Network Analytics (Stealthwatch)]
tags: [Stealthwatch]
---

Cisco Secure Network Analytics, formerly known as Stealthwatch, provides comprehensive visibility and security analytics for network traffic. It detects threats, accelerates threat response, and secures critical assets across various network environments.
> * **SMC (Stealthwatch Management Console)**: The central management and analysis point, it provides a unified interface for monitoring, alerting, and reporting on network activity and security events.
> * **Flow Collector**: This component gathers and stores network telemetry data (NetFlow, sFlow, etc.) from various sources, which is then analyzed for potential security threats and anomalies.
> * **Flow Sensor**: It monitors network traffic directly by capturing packets and generating flow records, using SPAN for network devices that do not have telemetry data (NetFlow, sFlow, etc.) capabilities.

![x](/static/2024-07-26-stealthwatch/00.png)

<br>

## Deploying SMC

Download the SMC iso installer and deploy it on the VMWare environment

![x](/static/2024-07-26-stealthwatch/01.png)

![x](/static/2024-07-26-stealthwatch/03.png)

<br>

Stealthwatch default users :

> * sysadmin/lan1cope
> * root/lan1cope
> * admin/lan411cope

<br>

Spin up the VM, login with sysadmin and give it IP Address

![x](/static/2024-07-26-stealthwatch/05.png)

<br>

Now the SMC Web UI is accessible, log in with the admin user

![x](/static/2024-07-26-stealthwatch/08.png)

<br>

Complete the initial setup, starting with changing the default passwords

![x](/static/2024-07-26-stealthwatch/09.png)

<br>

Next configure the Management Interface

![x](/static/2024-07-26-stealthwatch/10.png)

<br>

Then the hostname and domain, along with the IP Address range that the Stealthwatch will monitor within the environment

![x](/static/2024-07-26-stealthwatch/11.png)

<br>

After that configure the DNS and NTP

![x](/static/2024-07-26-stealthwatch/12.png)

![x](/static/2024-07-26-stealthwatch/13.png)

<br>

Lastly, enter the SMC IP Address which is the self IP Address, then finish the initial setup

![x](/static/2024-07-26-stealthwatch/14.png)

![x](/static/2024-07-26-stealthwatch/15.png)

<br>
<br>

## Deploying Flow Collector

Download the Flow Collector iso installer and deploy it

![x](/static/2024-07-26-stealthwatch/02.png)

![x](/static/2024-07-26-stealthwatch/04.png)

<br>

Spin up the VM and select the deployment with no Data Store, this configuration will store the telemetry data locally on the Flow Collector

![x](/static/2024-07-26-stealthwatch/06.png)

<br>

Then configure the IP Address

![x](/static/2024-07-26-stealthwatch/07.png)

<br>

After that, access the Flow Collector Web UI

![x](/static/2024-07-26-stealthwatch/16.png)

<br>

Then follow the initial setup as well

![x](/static/2024-07-26-stealthwatch/17.png)

![x](/static/2024-07-26-stealthwatch/18.png)

![x](/static/2024-07-26-stealthwatch/19.png)

![x](/static/2024-07-26-stealthwatch/20.png)

![x](/static/2024-07-26-stealthwatch/21.png)

<br>

Finally enter the SMC IP Address along with the Domain and the Collector Port

![x](/static/2024-07-26-stealthwatch/22.png)

<br>

Now back on the SMC >> Configure >> Central Management >> Inventory, we can see both SMC and Flow Connector are up and connected

![x](/static/2024-07-26-stealthwatch/23.png)

<br>

Here also we centrally manage all the configuration for Flow Collector

![x](/static/2024-07-26-stealthwatch/24.png)

<br>

## Configuring Netflow Traffic

On Configure >> Host Group Management, we can define the IP Address ranges as well as specific hosts that we want to monitor

![x](/static/2024-07-26-stealthwatch/25.png)

<br>

On the Cisco Router, we'll configure the router so it sends netflow traffic to Flow Connector

```shell
flow record SW_RECORD
 description netflow_to_stealthwatch
 match ipv4 tos
 match ipv4 protocol
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 match interface input
 match flow direction
 match flow cts source group-tag
 match flow cts destination group-tag
 collect routing source as
 collect routing destination as
 collect routing next-hop address ipv4
 collect ipv4 dscp
 collect ipv4 id
 collect ipv4 source prefix
 collect ipv4 source mask
 collect ipv4 destination mask
 collect ipv4 ttl minimum
 collect ipv4 ttl maximum
 collect transport tcp flags
 collect interface output
 collect counter bytes
 collect counter packets
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last
 collect application name
 collect application http url
 collect application http host
```

> This configuration creates a flow record named SW_RECORD designed to send NetFlow data to Stealthwatch. The flow record specifies the fields to be matched and collected for network traffic monitoring and analysis.

<br>

```shell
flow exporter SW_EXPORTER
 description export_stealthwatch
 destination 198.18.134.51
 source gigabitEthernet1
 transport udp 2055
 export-protocol ipfix
 template data timeout 30
 option interface-table
 option application-table timeout 10
```

> This configuration defines a flow exporter named SW_EXPORTER used to export flow records to Stealthwatch. The settings specify the destination, source interface, transport protocol, and options for exporting data.

<br>

```shell
flow monitor SW_MON
 exporter SW_EXPORTER
 cache timeout active 60
 cache timeout inactive 15
 record SW_RECORD
```

> Then this configuration defines a flow monitor named SW_MON, which uses the previously defined flow exporter and flow record to monitor and export network traffic data to Stealthwatch. The settings specify the exporter, cache timeouts, and record to be used.

<br>

```shell
interface gigabitEthernet1
 ip flow monitor SW_MON input
 ip nbar protocol-discovery

interface gigabitEthernet2
 ip flow monitor SW_MON input
 ip nbar protocol-discovery

interface gigabitEthernet6
 ip flow monitor SW_MON input
 ip nbar protocol-discovery
```

> This configuration applies the previously defined flow monitor (SW_MON) and enables NBAR (Network-Based Application Recognition) protocol discovery on interfaces gigabitEthernet1, gigabitEthernet2, and gigabitEthernet6. This setup ensures that network traffic on these interfaces is monitored and analyzed, with detailed flow data exported to Stealthwatch.

<br>

Run show flow to see the configuration

![x](/static/2024-07-26-stealthwatch/26.png)

<br>

Back on SMC, we can see the Flow Collection Trend rising up as the traffic is coming in to Flow Collector

![x](/static/2024-07-26-stealthwatch/27.png)

<br>

On Investigation >> Flow Search, we are able to see traffic as a one contextual data, even though the actual traffic traverses over multiple different network devices sending netflow to Flow Collector

![x](/static/2024-07-26-stealthwatch/28.png)

<br>
<br>


## Deploying Flow Sensor

Download the Flow Sensor iso installer and deploy it

![x](/static/2024-07-26-stealthwatch/29.png)

![x](/static/2024-07-26-stealthwatch/29a.png)

<br>

Spin up the VM configure the IP Address

![x](/static/2024-07-26-stealthwatch/30.png)

<br>

After it restarts access the Web UI

![x](/static/2024-07-26-stealthwatch/31.png)

<br>

Follow the initial config as usual, then point it to the SMC IP Address and select the Flow Collector

![x](/static/2024-07-26-stealthwatch/32.png)

<br>

Now the Flow Sensor is up and connected as well on SMC

![x](/static/2024-07-26-stealthwatch/33.png)

<br>

Access the Flow Sensor's Web UI and Enable ERSPAN Decapsulation, because we will configure ERSPAN on the Network Device

![x](/static/2024-07-26-stealthwatch/33a.png)

> ERSPAN (Encapsulated Remote Switched Port Analyzer) is a protocol used to mirror traffic from a source to a destination, encapsulating the mirrored packets in GRE (Generic Routing Encapsulation) and sending them over IP networks. Enabling ERSPAN  decapsulation on a flow sensor allows the sensor to process and analyze ERSPAN traffic. Decapsulation itself is the process of removing this encapsulation to access the original mirrored packets.

<br>

On the Network Device, enable ERSPAN to mirror traffic from a specified source interface and send it to Flow Sensor

```shell
monitor session 1 type erspan-source
 no shutdown
 source interface Gi3
 destination
  erspan-id 1
  ip address 198.18.134.52
  origin ip address 198.18.134.122
```

<br>

Run show monitor session to see the configuration

![x](/static/2024-07-26-stealthwatch/34.png)

<br>

This router handles traffic from 10.122.0.21 on the Gig3 interface to the Internet, this traffic should be mirrored and sent to the Flow Sensor

![x](/static/2024-07-26-stealthwatch/35.png)

<br>

Back on SMC on Investigation >> Interface >> fs (Flow Sensor), we can see it starting to pick up traffic

![x](/static/2024-07-26-stealthwatch/36.png)

<br>

Then on Investigation >> Flow Search, here we see the traffic on host 10.122.0.21 is indeed showing up

![x](/static/2024-07-26-stealthwatch/37.png)

<br>









