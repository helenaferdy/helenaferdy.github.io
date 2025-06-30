---
title: FortiAuthenticator
date: 2025-06-30 07w:00:00 +0700
categories: [Security, Fortigate]
tags: [Fortigate]
---

FortiAuthenticator is Fortinet’s centralized authentication management solution that provides secure identity and access services for enterprise networks. It supports user authentication through various methods like RADIUS, LDAP, SAML, and certificate-based login, and integrates with Fortinet devices (like FortiGate) to enforce user-based policies, enable SSO, and improve visibility and control over network access.

<br>

Once installed like any other Fortinet device, access the Web GUI

![x](/static/2025-06-30-fortiauthenticator/01.png)

<br>

## LDAP Integration

On Authentication >> Remote Auth Servers >> LDAP, create a new LDAP integration to Domain Controller

![x](/static/2025-06-30-fortiauthenticator/02.png)

<br>

Once configured correctly, we can now browse the domain grops and users

![x](/static/2025-06-30-fortiauthenticator/03.png)

<br>

Next hit Import Users, and import the LDAP Users based on the desired groups

![x](/static/2025-06-30-fortiauthenticator/04.png)

<br>

After that, the LDAP users are now available on User Manaement >> Remote Users

![x](/static/2025-06-30-fortiauthenticator/05.png)

<br>

Here we also add a local user 'localmaster' for testing purposes

![x](/static/2025-06-30-fortiauthenticator/06.png)

<br>

Next we create 2 new goups, one for Locaol Users and the other for LDAP Users, here we add the users that we'd like to include

![x](/static/2025-06-30-fortiauthenticator/07.png)

<br>

After that we create a new Realm to point to the integrated LDAP

![x](/static/2025-06-30-fortiauthenticator/08.png)

<br>

## Fortigate Integration

On FortiAuthenticator's intefrace, make sure its able to receive Radius connections from fortigate

![x](/static/2025-06-30-fortiauthenticator/16.png)

<br>

Then on RADIUS Service >> Clients, add Fortigate as the client

![x](/static/2025-06-30-fortiauthenticator/09.png)

<br>

Next create a Radius Policy, select the created Radius Client

![x](/static/2025-06-30-fortiauthenticator/10.png)

<br>

Here we just skip ahead because we dont configure any specific attributes

![x](/static/2025-06-30-fortiauthenticator/11.png)

<br>

For authentication type, we stick with default Password/OTP and we enable some EAP protocols incase needed in the future

![x](/static/2025-06-30-fortiauthenticator/12.png)

<br>

And here we add the created Realm, we configure so it looks up users on the 2 User Groups created earlier

![x](/static/2025-06-30-fortiauthenticator/13.png)

<br>

For the factor, we keep the default

![x](/static/2025-06-30-fortiauthenticator/14.png)

<br>

And lastly we can hit Save and Finish

![x](/static/2025-06-30-fortiauthenticator/15.png)

<br>

On the Fortigate side, we add a new Radius Server pointing to the FortiAuthenticator

![x](/static/2025-06-30-fortiauthenticator/17.png)

<br>

We can test the user credentials to make sure the integration is running

![x](/static/2025-06-30-fortiauthenticator/18.png)

<br>

On the FAC, we can see the success log for the tested user

![x](/static/2025-06-30-fortiauthenticator/19.png)

<br>

Next on Fortigate, create a new User Groups that points to the Radius Server created

![x](/static/2025-06-30-fortiauthenticator/20.png)

<br>

And on the Authentication Settings, we configure the Captive Portal to point to the same Radius Server, this portal will be used for users to enter their credentials

![x](/static/2025-06-30-fortiauthenticator/21.png)

<br>

Finally, we can use the User Groups on the Firewall Policy

![x](/static/2025-06-30-fortiauthenticator/22.png)

<br>

## User Testing

On the user end, when we try to access internet we will be redirected to the Captive Portal to authenticate

![x](/static/2025-06-30-fortiauthenticator/23.png)

<br>

Once successfully authenticate, user is now granted access

![x](/static/2025-06-30-fortiauthenticator/24.png)

![x](/static/2025-06-30-fortiauthenticator/25.png)

![x](/static/2025-06-30-fortiauthenticator/26.png)

<br>










