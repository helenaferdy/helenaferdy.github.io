---
title: Syslog Server with Python
date: 2023-07-19 11:30:00 +0700
categories: [Python, Syslog]
tags: [Syslog]
---

A syslog server is a central logging server that gathers and stores log messages sent by various devices, applications, and systems across a network. <br>
The syslog protocol is a standard method used for sending log messages over an IP network.
<br><br>
Here we're gonna create our own Syslog Server using Python.

<br>
<br>

# The Python Code

I'm not gonna explain too much about the code, i stole it straight from ChatGPT. <br>
The bottom line is this code will serve your server as a Syslog Server and parse the Cisco log messages or any log messages and save it into a log file.

```python
#!/usr/bin/env python3

import socket
import time

def save_to_log(log_data):
    with open('/opt/python-syslog/output-syslog.log', 'a') as log_file:
        log_file.write(log_data)

def parse_syslog_data(data, ipadd):
    data = f"{data}: {ipadd}\n"
    return data

def syslog_server(host, port):
    try:
        server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        server_socket.bind((host, port))
        print(f"Syslog server listening on {host}:{port}")

        while True:
            data, address = server_socket.recvfrom(4096)
            log_data = data.decode('utf-8')
            parsed_log = parse_syslog_data(log_data, address[0])
            print(parsed_log)
            save_to_log(parsed_log)

    except Exception as e:
        print(f"Error: {e}")
    finally:
        server_socket.close()

if __name__ == "__main__":
    syslog_server('198.18.0.201', 514)
```
> change the **with open('<span style="color:yellow">/opt/python-syslog/output-syslog.log</span>', 'a') as log_file:** with your own path. <br>
> change the **syslog_server('<span style="color:yellow">198.18.0.201</span>', 514)** with your server's IP Address.

<br>


Save it and try running the code, it'll start listening on port 514

```shell
sudo python3 python-syslog.py
```

```shell
helena@ubuntuy:~/syslog$ sudo python3 python-syslog.py 
Syslog server listening on 198.18.0.201:514
```

<br>

Now set the target device (eg: cisco routers) to send syslog messages to our syslog server that we just set up.

```
xe4#conf t
xe4(config)#logging host 198.18.0.201
```

<br>

After a while, you'll start seeing syslog messages being printed on the console

```
Syslog server listening on 198.18.0.201:514

198.18.0.124: <189>225: *Jul 19 06:55:12.582: %SYS-5-CONFIG_I: Configured from console by helena on vty0 (10.16.36.109)
198.18.0.124: <189>226: *Jul 19 06:55:35.935: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback2, changed state to up
198.18.0.124: <189>227: *Jul 19 06:55:42.307: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback2, changed state to down
198.18.0.124: <189>228: *Jul 19 06:55:42.307: %LINK-5-CHANGED: Interface Loopback2, changed state to administratively down
```
<br>

And a new log file will be created containing the logs

```shell
helena@ubuntuy:~/syslog$ cat output-syslog.log 

<189>225: *Jul 19 06:55:12.582: %SYS-5-CONFIG_I: Configured from console by helena on vty0 (10.16.36.109)
<189>226: *Jul 19 06:55:35.935: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback2, changed state to up
<189>227: *Jul 19 06:55:42.307: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback2, changed state to down
<189>228: *Jul 19 06:55:42.307: %LINK-5-CHANGED: Interface Loopback2, changed state to administratively down
```

<br>
<br>

# Running the code as a Linux Service

After we verify that the code is working well, now we're gonna make it to run as a service on our server.
<br>

First, move the python code to a secure and neat directory, here we'll use /opt/python-syslog, and give it executable permission

```shell
sudo mkdir /opt/python-syslog
sudo cp python-syslog.py opt/python-syslog/python-syslog.py
sudo chmod +x opt/python-syslog/python-syslog.py
```

>PS: Optionally, modify the code to also save the log in this directory if it is not already

<br>

After that, create the service file

```shell
sudo nano /etc/systemd/system/python-syslog.service
```

Paste in these lines

```
[Unit]
Description=Python Syslog Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/python-syslog/
ExecStart=/opt/python-syslog/python-syslog.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

<br>

Now we can run this code just like any other linux services.

```
sudo systemctl daemon-reload
sudo systemctl start python-syslog
sudo systemctl enable python-syslog
```

<br>

Check the service status or view its logs by running these

```shell
systemctl status python-syslog
journalctl -u python-syslog
```

```
● python_syslog.service - Python Syslog Service
     Loaded: loaded (/etc/systemd/system/python_syslog.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-07-19 06:49:41 UTC; 28min ago
   Main PID: 23763 (python3)
      Tasks: 1 (limit: 4557)
     Memory: 3.7M
        CPU: 153ms
     CGroup: /system.slice/python_syslog.service
             └─23763 python3 /opt/python-syslog/python-syslog.py


Jul 19 06:49:41 ubuntu2 systemd[1]: Started Python Syslog Service.
Jul 19 06:54:50 ubuntu2 python-syslog.py[23763]: Syslog server listening on 198.18.0.201:514
Jul 19 06:54:50 ubuntu2 python-syslog.py[23763]: 198.18.0.125: <188>1820499: *Jul 19 06:41:06.923: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from back>
```

<br>

And finally run this command to watch as the logs piling up in your output file

```shell
tail -f /opt/python-syslog/output-syslog.log 
```

```
<188>1820764: *Jul 19 07:10:19.143: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 2.2.34.4, GigabitEthernet2
<188>1820765: *Jul 19 07:10:28.426: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 2.2.34.4, GigabitEthernet4
<188>1820766: *Jul 19 07:10:33.584: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 2.2.12.1, GigabitEthernet2
<188>1820767: *Jul 19 07:10:39.444: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 2.2.45.4, GigabitEthernet2
<188>1820768: *Jul 19 07:10:47.498: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 2.2.34.4, GigabitEthernet2
<188>1820769: *Jul 19 07:10:57.350: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 2.2.34.4, GigabitEthernet2
<188>1820770: *Jul 19 07:11:07.145: %OSPF-4-ERRRCV: Received invalid packet: mismatched area ID from backbone area from 2.2.34.4, GigabitEthernet4
```
