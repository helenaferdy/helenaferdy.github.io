---
title: Solarwinds Server & Application Monitor
date: 2023-10-13 13:30:00 +0700
categories: [Monitoring Tools, Solarwinds]
tags: [Solarwinds]
---

<br>

SolarWinds Server & Application Monitor (SAM) is a software tool for monitoring and managing the performance and availability of servers and applications in IT environments. It provides real-time monitoring of server metrics like CPU and memory usage and application performance.

<br>
<br>

## Installing Solarwinds

Download the installer on [Solarwinds](https://www.solarwinds.com/server-application-monitor) website

![x](/static/2023-10-13-solarwinds/00.png)

<br>

Then run the installer on Windows Machine

![x](/static/2023-10-13-solarwinds/01.png)

<br>

Select the installation type

![x](/static/2023-10-13-solarwinds/02.png)

<br>

Select the SQL and Destination folder

![x](/static/2023-10-13-solarwinds/03.png)

<br>

Then after system check is passed, choose install

![x](/static/2023-10-13-solarwinds/04.png)

<br>

On Database setting, point it to our database and provide the credentials

![x](/static/2023-10-13-solarwinds/06.png)

![x](/static/2023-10-13-solarwinds/07.png)

![x](/static/2023-10-13-solarwinds/08.png)

<br>

For the management web, select the IP Address and the port

![x](/static/2023-10-13-solarwinds/09.png)

<br>

Then select database for logging

![x](/static/2023-10-13-solarwinds/10.png)

![x](/static/2023-10-13-solarwinds/11.png)

<br>

Next configure the database for Netflow Traffic Analyzer

![x](/static/2023-10-13-solarwinds/12.png)

![x](/static/2023-10-13-solarwinds/13.png)

<br>

Review the setup and hit install

![x](/static/2023-10-13-solarwinds/14.png)

<br>

The installation will finish after a couple minutes

![x](/static/2023-10-13-solarwinds/15.png)

<br>
<br>

## Accessing Solarwinds

Now the management web should be accessible using the IP Address / FQDN provided

![x](/static/2023-10-13-solarwinds/16.png)

<br>

Configure the Network Sonar Wizard to discover the devices

![x](/static/2023-10-13-solarwinds/17.png)

<br>

And now the solarwinds should be fully operational

![x](/static/2023-10-13-solarwinds/18.png)

![x](/static/2023-10-13-solarwinds/19.png)

![x](/static/2023-10-13-solarwinds/20.png)

![x](/static/2023-10-13-solarwinds/21.png)

![x](/static/2023-10-13-solarwinds/22.png)

<br>





















