---
title: SSL Offloading on F5
date: 2023-08-05 08:30:00 +0700
categories: [Security, F5]
tags: [F5]
---

<br>

## What is SSL Offloading?

<br>

F5 SSL offloading is the practice of transferring the task of encrypting and decrypting secure web connections from web servers to specialized F5 devices, enhancing performance by freeing up server resources and improving security management.

<br>

![x](/static/2023-08-05-f5-ssl-offload/01.png)

In this setup, there will be 2 unsecure web servers running on port 80 being members of a Virtual Server which where the SSL Offloading takes place. <br>


<br>
<br>

## Configure Certificates

<br>

First let's update F5's certificate so it'll use a CA Signed Certificate. <br>
Go to System >> Certificate Management >> Traffic Certificate Management, Create new


![x](/static/2023-08-05-f5-ssl-offload/02.png)

<br>

After that we'll get a Certificate Signing Request (CSR)

![x](/static/2023-08-05-f5-ssl-offload/03.png)

<br>

Then just sign the CSR on the Certificate Authority (CA) of the local domain, in this case is on a Windows Server of the Domain Controller.

![x](/static/2023-08-05-f5-ssl-offload/04.png)

<br>

After that, just import the certificate that we just got from the CA 


![x](/static/2023-08-05-f5-ssl-offload/05.png)

![x](/static/2023-08-05-f5-ssl-offload/06.png)

<br>
<br>


## Configure SSL Profile

<br>

Now we create a SSL Profile that we'll to use on the Virtual Server, <br>
Go to Local Traffic >> Profiles >> SSL, Create new

![x](/static/2023-08-05-f5-ssl-offload/08.png)

![x](/static/2023-08-05-f5-ssl-offload/09.png)

Just select the certificate and the key that has just been created earlier

<br>
<br>

## Configure SSL Offloading on Virtual Server

<br>

Now go to the LTM's Virtual Server, change the port to 443 and add the SSL Profile

![x](/static/2023-08-05-f5-ssl-offload/10.png)

![x](/static/2023-08-05-f5-ssl-offload/11.png)

<br>

And now if we access the Virtual Server on https://f5.helena.gg, we should get a secure SSL connection

![x](/static/2023-08-05-f5-ssl-offload/12.png)


<br>