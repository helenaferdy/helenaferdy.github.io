---
title: ASA AnyConnect Certificate Authentication
date: 2025-03-25 09:00:00 +0700
categories: [Security, Cisco Adaptive Security Appliance (ASA)]
tags: [ASA]
---

Cisco ASA AnyConnect VPN Certificate Authentication enhances security by providing strong, encrypted authentication while reducing reliance on passwords. It can be used as the sole authentication method for a seamless, passwordless experience or combined with password-based authentication for added security through integration with Cisco ISE or Cisco Duo.


## ASA Certificate

First we need to add the Root CA Certificate to ASA, on Configuration >> Remote Access VPN >> CA Certificates, add the Root CA

![x](/static/2025-03-25-asa-vpn-certificate/04.png)

![x](/static/2025-03-25-asa-vpn-certificate/05.png)

<br>

Then we create a CSR named for the Identity Certificate "VPN-Certificate", click add and fill in the details

![x](/static/2025-03-25-asa-vpn-certificate/06.png)

<br>

And export the CSR

![x](/static/2025-03-25-asa-vpn-certificate/07.png)

<br>

Sign the CSR on the Certificate Authority (CA) Server

![x](/static/2025-03-25-asa-vpn-certificate/08.png)

<br>

Then install the certificate

![x](/static/2025-03-25-asa-vpn-certificate/09.png)

![x](/static/2025-03-25-asa-vpn-certificate/10.png)

<br>

Finally attach the certificate to the outside interface

![x](/static/2025-03-25-asa-vpn-certificate/11.png)

```
ssl trust-point VPN-Certificate outside
```

<br>

## ASA VPN

Here we'll modify an existing Connection Profile to use Certificate Authentication, all needs to be done is changing the Authentication Method to "Certificate Only"

![x](/static/2025-03-25-asa-vpn-certificate/13.png)

![x](/static/2025-03-25-asa-vpn-certificate/12.png)

<br>

And on Advanced >> Authentication, select the UPN to be used as client's username

![x](/static/2025-03-25-asa-vpn-certificate/14.png)

<br>


## Client Certificate

On the client PC thats part of the domain, on Personal Certificates, create new User Certificate

![x](/static/2025-03-25-asa-vpn-certificate/01.png)

![x](/static/2025-03-25-asa-vpn-certificate/02.png)

<br>

The newly made certificate should be present on the Personal Certificates directory now

![x](/static/2025-03-25-asa-vpn-certificate/03.png)

<br>

Now if we try accessing the ASA's VPN Gateway, we should be asked to use our User Certificate

![x](/static/2025-03-25-asa-vpn-certificate/15.png)

<br>

## VPN Authentication with Certificate

Now when we try connecting to VPN, we can see that certificate will be used for authentication

![x](/static/2025-03-25-asa-vpn-certificate/16.png)

<br>

And after successfully connecting, the certificate authentication details can be seen on ASA

![x](/static/2025-03-25-asa-vpn-certificate/17.png)

<br>

## VPN Authentication with Certificate and AAA

Other than using Certificate Only, we can also use it on top of password-based authentication, for example here we'll also use ISE

![x](/static/2025-03-25-asa-vpn-certificate/18.png)

<br>

When we connect to the VPN, the certificate will be validated first before presenting the username and password form

![x](/static/2025-03-25-asa-vpn-certificate/19.png)

![x](/static/2025-03-25-asa-vpn-certificate/20.png)

<br>

## Invalid Certificate

If we try connecting without having a User Certificate, we'll get an invalid certificate error

![x](/static/2025-03-25-asa-vpn-certificate/21.png)

<br>

The not so cool thing is, if we export the User Certificate (along with the key) from one PC and Import it to the other PC with different user, and that PC is not even part of the domain, we will be allowed to authenticate again

![x](/static/2025-03-25-asa-vpn-certificate/22.png)

<br>


