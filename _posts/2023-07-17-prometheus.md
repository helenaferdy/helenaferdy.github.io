---
title: Monitoring Tools with Prometheus
date: 2023-07-17 13:30:00 +0700
categories: [Prometheus]
tags: [network automation, prometheus, monitoring, linux, node exporter, snmp exporter]
---

## What is Prometheus?

Prometheus is an open-source systems monitoring and alerting toolkit. It is widely used for monitoring the performance and health of computer systems and applications in a distributed environment. Prometheus follows a pull-based model, where it periodically scrapes metrics data from configured targets, such as servers, services, and containers.

<br>

## Installing Prometheus on Linux

First, let's create a service user named prometheus

```bash
sudo useradd --no-create-home -rs /bin/false prometheus
```
> the <span style="color:yellow">*--no-create-home -rs /bin/false* </span> ensures this user doesn't have a home directory and cannot be used to login.

<br>

Then create directories for Prometheus' configuration files

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

<br>

change the owner of those directories
```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

<br>

Then download the [Prometheus file](https://prometheus.io/download/), you can use *wget* to do this on linux

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
```

<br>

After that extract it

```bash
tar xvzf prometheus-2.45.0.linux-386
```

<br>

Next we need to copy the files to its respective configuration directories and 

```bash
cd prometheus-2.45.0.linux-386/
sudo cp prometheus promtool /usr/local/bin/
sudo cp prometheus.yml /etc/prometheus/
sudo cp -r consoles console_libraries /etc/prometheus/
```

<br>

Then open the prometheus.yml file

```bash
sudo nano /etc/prometheus/prometheus.yml
```

<br>

make sure it contains this configuration
```yml
# my global config
global:
scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
# scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
alertmanagers:
- static_configs:
- targets:
# - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
# - "first_rules.yml"
# - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
# The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
- job_name: "prometheus"

# metrics_path defaults to '/metrics'
# scheme defaults to 'http'.

static_configs:
- targets: ["localhost:9090"]
```

<br>

Next we create system service file

```bash
sudo nano /etc/systemd/system/prometheus.service
```

<br>

Add this to the file
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

<br>

And that should be all. Now we start the service

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

<br>

Check the service status with this command
```bash
systemctl status prometheus
```
```
helena@ubuntuhelena:~$ systemctl status prometheus
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-07-17 05:56:39 UTC; 46min ago
   Main PID: 4171 (prometheus)
      Tasks: 14 (limit: 9386)
     Memory: 31.7M
        CPU: 16.200s
     CGroup: /system.slice/prometheus.service
             └─4171 /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ -->
```

<br>

If it's active and running, you can access it with browser on http://<the-ip-address:9090>

![01](/static/2023-07-17-prometheus/01.png)

<br>

To check the status of your node, go to **Status > Targets**.

![02](/static/2023-07-17-prometheus/02.png)

