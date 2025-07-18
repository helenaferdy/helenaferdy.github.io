---
title: Check Point Sandblast
date: 2024-09-03 09:00:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

Check Point Threat Emulation SandBlast is a security solution that proactively defends against advanced threats by simulating and analyzing potentially malicious files in a secure, isolated environment (sandbox). This technology detects and prevents zero-day attacks, ransomware, and other sophisticated malware by executing suspicious files in a virtual environment to observe their behavior before they can impact the network.

![x](/static/2024-09-03-checkpoint-sandblast/00.png)

<br>

## Enabling Threat Emulation

In this deployment, we added a freshly installed Security Gateway that we'll use as the Sandblast Box (cpsb)

![x](/static/2024-09-03-checkpoint-sandblast/01.png)

<br>

Before enabling Sandblast, make sure this box can access internet

![x](/static/2024-09-03-checkpoint-sandblast/02.png)

<br>

Now on cpsb, enable Threat Emulation and select the location to be local

![x](/static/2024-09-03-checkpoint-sandblast/03.png)

<br>

Hit next until it finishes, after that we have Threat Emulation Blade enabled on this box

![x](/static/2024-09-03-checkpoint-sandblast/04.png)

<br>

Next on the Security Gateway (cpsg), enable the Threat Emulation as well and point it to other box

![x](/static/2024-09-03-checkpoint-sandblast/05.png)

<br>

And select the cpsb box

![x](/static/2024-09-03-checkpoint-sandblast/06.png)

<br>

Now the Security Gateway also has Threat Emulation enabled, but pointing to a remote box

![x](/static/2024-09-03-checkpoint-sandblast/07.png)

<br>

## Configuring Policy

On Threat Prevention >> Custom Policy, create a new Profile and enable Threat Emulation Blade

![x](/static/2024-09-03-checkpoint-sandblast/08.png)

<br>

Enable the protocols which we'd like to send to sandbox, in this case we'll focus on SMB

![x](/static/2024-09-03-checkpoint-sandblast/09.png)

<br>

Then on the File Types, we can see all the supprted types for sandboxing

![x](/static/2024-09-03-checkpoint-sandblast/10.png)

<br>

## Emulating Threat

To emulate threat, we'll download infected files from [Eicar](https://www.eicar.org) and save it on our file server

![x](/static/2024-09-03-checkpoint-sandblast/11.png)

<br>

Here on the user PC, first we try copying normal files over SMB

![x](/static/2024-09-03-checkpoint-sandblast/12.png)

<br>

On the Sandblast log we can see the files are determined to be clean and allowed

![x](/static/2024-09-03-checkpoint-sandblast/13.png)

![x](/static/2024-09-03-checkpoint-sandblast/14.png)

<br>

But when we try copying infected files, we get a network error

![x](/static/2024-09-03-checkpoint-sandblast/15.png)

<br>

That's because Check Point determines that the files are not safe and take the prevent action

![x](/static/2024-09-03-checkpoint-sandblast/16.png)

![x](/static/2024-09-03-checkpoint-sandblast/17.png)

<br>

## Threat Extraction

Check Point Threat Extraction is a cybersecurity solution designed to protect against document-based threats by removing potentially malicious content from files. It proactively cleans files by extracting and reconstructing them, stripping away exploitable content, such as embedded macros or active code, while preserving the original format. This ensures that only safe, sanitized files are delivered to users, reducing the risk of zero-day attacks and malware infections.

![x](/static/2024-09-03-checkpoint-sandblast/18.png)

<br>

On the Client PC, if we try accessing malicious files like this [Test File](https://sophostest.com/Sandstorm/SBTestFile1.pdf), we will still be able to download it like normal

![x](/static/2024-09-03-checkpoint-sandblast/19.png)

<br>

But behind the hood, Check Point runs an extraction to remove any potensially malicious threat inside the file, and delivering the cleaned up version of it 

![x](/static/2024-09-03-checkpoint-sandblast/20.png)

![x](/static/2024-09-03-checkpoint-sandblast/21.png)

<br>









