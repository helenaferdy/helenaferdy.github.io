---
title: Python Network Automation with Netmiko and NTC Templates
date: 2023-07-16 14:30:00 +0700
categories: [Python Automation]
tags: [ntc templates, netmiko]
---


In today's network infrastructure, automation plays a crucial role in managing and configuring devices efficiently. Netmiko, a multi-vendor library, and NTC Templates, a collection of parsing templates, are powerful tools that can simplify the process of obtaining and manipulating data from network devices. In this article, we will explore how to leverage Netmiko and NTC Templates to streamline network automation tasks.

## What's Netmiko?
> An open-source library that provides a simplified and consistent interface for network device interactions over SSH.

## What's NTC Templates?
> A collection of structured data parsing templates designed to extract valuable information from the command output of various network devices.

<br>
<br>

## Setting Up Netmiko
First of all, you need to make sure you have Netmiko installed on your machine with this command
```shell
pip install netmiko
```
<br>

Then import the netmiko module
```python
from netmiko import ConnectHandler
```
<br>

After that, initialize the device details
```python
device = {
    'device_type': 'cisco_ios',    # Specify the device type
    'ip': '198.18.0.121',           # IP address of the device
    'username': 'cisco',   # SSH username
    'password': 'cisco',   # SSH password
    'secret': 'cisco' # Enable secret/password
}
```
<br>

And then we connect to the device, enter enable mode and send the command with this code
```python
net_connect = ConnectHandler(**device)
net_connect.enable()
output = net_connect.send_command("show cdp neighbors")
```
<br>

Finally run this code to print the output
```python
print(output)
```

```
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone, 
                  D - Remote, C - CVTA, M - Two-port Mac Relay 

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
xe5              Gig 6             152              R I   CSR1000V  Gig 1
xe4              Gig 6             136              R I   CSR1000V  Gig 1
xe3              Gig 6             157              R I   CSR1000V  Gig 1
xe2              Gig 6             173              R I   CSR1000V  Gig 1
UCSM-A(SSI17360D2Y)
                 Gig 6             170             S I C  UCS-FI-62 Vethernet867
xr1              Gig 6             162               R    IOS-XRv 9 Gig 0/0/0/0
```
<br>
<br>

## Setting Up NTC Templates
Using only netmiko, all you get is a text output, which is fine if it's all you need. But if you need to cook the data or to get specific data from the output, it will be easier to do if the output is in a json format instead. <br>
This is where NTC Templates comes in to parse the raw output to become json.

<br>
First, install the module

```shell
pip install ntc-templates
```
<br>

Then import the module
```python
from ntc_templates.parse import parse_output
```
<br>

From the previous code above, we already have the text output saved in a *output* variable, now we pass that variable in the code below to parse it into json

```python
parsed_output = parse_output(platform=device['device_type'], command="show cdp neighbors", data=output)
```
<br>

And lastly print the final output.
```python
print(parsed_output)
```

```json
[
  {
    'neighbor': 'xe5',
    'local_interface': 'Gig 6',
    'capability': 'R I',
    'platform': 'CSR1000V',
    'neighbor_interface': 'Gig 1'
  },
  {
    'neighbor': 'xe4',
    'local_interface': 'Gig 6',
    'capability': 'R I',
    'platform': 'CSR1000V',
    'neighbor_interface': 'Gig 1'
  },
  {
    'neighbor': 'xe3',
    'local_interface': 'Gig 6',
    'capability': 'R I',
    'platform': 'CSR1000V',
    'neighbor_interface': 'Gig 1'
  },
  {
    'neighbor': 'xe2',
    'local_interface': 'Gig 6',
    'capability': 'R I',
    'platform': 'CSR1000V',
    'neighbor_interface': 'Gig 1'
  },
  {
    'neighbor': 'UCSM-A(SSI17360D2Y)',
    'local_interface': 'Gig 6',
    'capability': 'S I C',
    'platform': 'UCS-FI-62',
    'neighbor_interface': 'Vethernet867'
  },
  {
    'neighbor': 'xr1',
    'local_interface': 'Gig 6',
    'capability': 'R',
    'platform': 'IOS-XRv',
    'neighbor_interface': '9 Gig 0/0/0/0'
  }
]
```
<br>

Now we can easily cherry pick the data, for example if you want to get the name, platform and interface of the first neighbor, you can do this

```python
print(parsed_output[0]["neighbor"])
print(parsed_output[0]["platform"])
print(parsed_output[0]["neighbor_interface"])
```

and you'll get this output

```
xe5
CSR1000V
Gig 1
```