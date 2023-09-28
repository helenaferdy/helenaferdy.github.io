---
title: Github Pages Custom Domain
date: 2023-09-28 07:30:00 +0700
categories: [Other, Github Pages]
tags: [github pages]
---

<br>

## What is Github Pages Custom Domain?

GitHub Pages is a web hosting service provided by GitHub for hosting static websites directly from a GitHub repository. <br>
GitHub Pages Custom Domain allows us to use our own domain (e.g., helenaferdy.site) instead of the default GitHub Pages URL (e.g., helenaferdy.github.io).

<br>
<br>

## Configuring Custom Domain

First buy the domain the we'd like to use, here we buy a cheap domain of [helenaferdy.site](https://helenaferdy.github.io) for *Rp. 14.900* (*USD. 1$*)

![x](/static/2023-09-28-github-domain/00.png)

<br>

After completing payments and filling up some forms, the domain is up and active

![x](/static/2023-09-28-github-domain/01.png)

<br>

Now go to the Github Repository >> Settings >> Pages, enter the custom domain

![x](/static/2023-09-28-github-domain/02.png)

<br>

Next following the [Github Docs Regarding Custom Domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site), we have to configure the apex domain and www subdomain on the domain's DNS configuration

![x](/static/2023-09-28-github-domain/03.png)

![x](/static/2023-09-28-github-domain/03a.png)

<br>

First we configure the A and AAAA records

![x](/static/2023-09-28-github-domain/04.png)

> A (ipv4) and AAAA (ipv6) records are types of DNS records used to map domain names to IP addresses

<br>

After that we configure the CNAME record

![x](/static/2023-09-28-github-domain/05.png)

> A CNAME (Canonical Name) record is DNS record used to create an alias or redirection from one domain name to another

<br>

Now if we check on Github Repository, it shows that the site is hosted on the custom domain

![x](/static/2023-09-28-github-domain/08.png)

<br>

And accessing [helenaferdy.site](https://helenaferdy.github.io) shows that the custom domain is now up and accessible

![x](/static/2023-09-28-github-domain/06a.png)

<br>

Bonus, if we do a whois lookup, it shows the registration status of the domain 

![x](/static/2023-09-28-github-domain/07.png)

<br>




