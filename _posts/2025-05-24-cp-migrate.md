---
title: Check Point Policy Migration
date: 2025-05-24 07:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Migrating firewall policies to Check Point can follow two main paths: **Check Point-to-Check Point** migration, typically using the **Migration Tool** outlined in [sk180923](https://support.checkpoint.com/results/sk/sk180923), which ensures smooth transfer of configuration between versions or appliances. For migrations from **other firewall vendors**, Check Point offers the **SmartMove tool** described in [sk115416](https://support.checkpoint.com/results/sk/sk115416), enabling automated conversion of rules from platforms like Cisco ASA, Fortinet, and Palo Alto into Check Point format.

<br>

## Between Check Point

First download the 2 required files stated on the SK and copy them to the Management Server.

![x](/static/2025-05-24-cp-migrate/00.png)

<br>

Here we put them on '/home/admin/cp' directory along with the exported policy files, then configure the environment variables to point to the python sdk

```
export PYTHONPATH=$PYTHONPATH:/home/admin/cp/cp_mgmt_api_python_sdk-master/
```

![x](/static/2025-05-24-cp-migrate/01.png)

<br>

Next we run the tool with additional parameter for the tool to skip all the existing objects

```
python3 ExportImportPolicyPackage-master/import_export_package.py --skip-duplicate-objects True, select import and point it to the exported policy
```

![x](/static/2025-05-24-cp-migrate/02.png)

<br>

Next all we need to change is the custom name for the imported package, and select hit run

![x](/static/2025-05-24-cp-migrate/03.png)

![x](/static/2025-05-24-cp-migrate/04.png)

<br>

Now all the objects, policies, and NAT have been imported as a new policy package

![x](/static/2025-05-24-cp-migrate/05.png)

<br>

## From Different Firewall

First download 1 file from the SK named 'Check Point SmartMove Tool' and place it on our windows machine along with exported policy

![x](/static/2025-05-24-cp-migrate/05a.png)

<br>

Run the tool, select the vendor source, and point it to the exported policy

![x](/static/2025-05-24-cp-migrate/06.png)

<br>

After running the tool, we will get a bunch of files, some of them are the reports of the convert results

![x](/static/2025-05-24-cp-migrate/07.png)

<br>

This one shows all the successfully converted objects

![x](/static/2025-05-24-cp-migrate/08.png)

<br>

This one for NAT rules

![x](/static/2025-05-24-cp-migrate/09.png)

<br>

This one for the policy rules

![x](/static/2025-05-24-cp-migrate/10.png)

<br>

And this one, which is also for policy rules but optimized in a way that check point thinks is more organized and tidy

![x](/static/2025-05-24-cp-migrate/11.png)

<br>

Next copy the xnet1_object.sh and xnet1_policy.sh to the management server, run dos2unix to convert DOS/Windows format to Unix format, and give 777 permssion

```
dos2unix xnet1_object.sh xnet1_policy.sh
chmod 777 xnet1_object.sh xnet1_policy.sh
```

![x](/static/2025-05-24-cp-migrate/12.png)

<br>

And finally, we run the script, starting with the object one

![x](/static/2025-05-24-cp-migrate/13.png)

![x](/static/2025-05-24-cp-migrate/14.png)

<br>

And then the policy one

![x](/static/2025-05-24-cp-migrate/15.png)

![x](/static/2025-05-24-cp-migrate/16.png)

<br>

On SmartConsole, we will see the imported objects, NAT, and policy rules have been imported as a new policy package

![x](/static/2025-05-24-cp-migrate/17.png)

<br>





















