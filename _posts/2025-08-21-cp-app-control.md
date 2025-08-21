---
title: Check Point App Control & Content Awareness
date: 2025-08-21 07:30:00 +0700
categories: [Security, Check Point]
tags: [Check Point]
---

**URL Filtering:** This blade controls access to websites by sorting them into categories, allowing us to easily block malicious or inappropriate sites to enforce our web usage policy.

**Application Control:** This feature provides granular control by identifying specific applications and their functions, letting us decide not just *if* a user can access a site, but *what* they are allowed to do there, like blocking a file upload while still permitting browsing.

**Content Awareness:** Acting as a Data Loss Prevention (DLP) engine, this blade inspects the actual data within files and traffic, blocking the transfer of any information that matches predefined sensitive content like confidential keywords or specific file types.

## URL Filtering

Lets start off with URL Filtering, to do that we enable URL Filtering & Application Control Blades

![x](/static/2025-08-21-cp-app-control/01.png)

<br>

Next we need to enable both Blades on the Policy Package, this will allow us to use the features on each policy rule

![x](/static/2025-08-21-cp-app-control/02.png)

<br>

Then we create the policy, here we set up a simple URL Filtering policy where users are not allowed to access proxy sites in a pre-made bundle called Anonymizer

![x](/static/2025-08-21-cp-app-control/03.png)

<br>

And when we test trying to access proxy sites, the connection will get blocked

![x](/static/2025-08-21-cp-app-control/04.png)

<br>

On the logs we can see the traffic was blocked by the URL Filtering Blade

![x](/static/2025-08-21-cp-app-control/05.png)

<br>

## Application Control

Application Control lets us control the traffic even more granularly, for example instead of blocking the whole site, we will only block a upload operation, allowing users to browse and download but not not upload. To do this, Firewall needs to be able to inspect into the encrypted traffic over https using [HTTPS Inspection Blade](https://helenaferdy.github.io/posts/checkpoint-https-inspection/).

<br>

After that, we can set up a simple policy to block a specific upload operation of dropbox web

![x](/static/2025-08-21-cp-app-control/06.png)

<br>

Now users can still access dropbox but any upload operation will result in a failure

![x](/static/2025-08-21-cp-app-control/07.png)

<br>

On the logs side, we can see the traffic was dropped by the Application Control Blade

![x](/static/2025-08-21-cp-app-control/08.png)

<br>

## Content Awareness

Content Awareness give ever more granular control over the allowed traffic, instead of blocking an entire upload operation of a site, we can set this to only block a certain data type or content type of a file, forcing certain files to never leave the network while allowing others. To do this, first we enable the blade

![x](/static/2025-08-21-cp-app-control/09.png)

<br>

We also enable the blade on the policy package

![x](/static/2025-08-21-cp-app-control/10.png)

<br>

Then we can set up a simple rule to block any upload operation of microsoft word documents

![x](/static/2025-08-21-cp-app-control/11.png)

<br>

Now if we try uploading any docx files, it will be blocked while other file types are allowed

![x](/static/2025-08-21-cp-app-control/12.png)

<br>

The logs also show this block is the work of Content Awareness Blade

![x](/static/2025-08-21-cp-app-control/13.png)

<br>

Other than file type, we can also control files based on their content, hence the name content awareness, here we set up a rule to block any files that contains name 'Top_Secret'

![x](/static/2025-08-21-cp-app-control/14.png)

<br>

Now any files with name Top_Secret will also get dropped

![x](/static/2025-08-21-cp-app-control/15.png)

![x](/static/2025-08-21-cp-app-control/16.png)

<br>




























