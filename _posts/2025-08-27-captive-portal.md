---
title: Palo Alto Captive Portal
date: 2025-08-27 07:30:00 +0700
categories: [Security, Palo Alto]
tags: [Palo Alto]
---

Palo Alto Captive Portal is a web-based authentication method that forces unauthenticated users to log in before gaining network access. When a user initiates traffic, the firewall intercepts it and redirects the user to a login page, where credentials are validated against an authentication source (like LDAP or local DB).


## Configuring Captive Portal

First we'll setup a portal that authenticate using local users, here we create a new user for that

![x](/static/2025-08-27-captive-portal/01.png)

<br>

Next create a new Authentication Profile for Local Database that allows User1 to authenticate

![x](/static/2025-08-27-captive-portal/02.png)

![x](/static/2025-08-27-captive-portal/03.png)

<br>

Then configure the Authentication Portal to use the Local Auth Profile

![x](/static/2025-08-27-captive-portal/04.png)

<br>

After that, create a new Authentication Policy Rule to enforce authentication though default web form

![x](/static/2025-08-27-captive-portal/06.png)

<br>

And lastly, on the Inside Zone enable User Identification

![x](/static/2025-08-27-captive-portal/05.png)

<br>

Now on the client side, on the first time connecting to the internet they'll be redirected to login on the portal using local users

![x](/static/2025-08-27-captive-portal/07.png)

![x](/static/2025-08-27-captive-portal/08.png)

<br>

Logs on the firewall side verify that the authentication is working

![x](/static/2025-08-27-captive-portal/09.png)

![x](/static/2025-08-27-captive-portal/10.png)

<br>

We can run 'show user ip-user-mapping all' to see the mapped users

![x](/static/2025-08-27-captive-portal/10a.png)

<br>

And running 'debug user-id reset captive-portal ip-address' allows us to deautheticate specific user

![x](/static/2025-08-27-captive-portal/10b.png)

<br>

## Configuring LDAP

To allow clients to use LDAP Users on the portal, first we create or use existing LDAP Server Profile

![x](/static/2025-08-27-captive-portal/11.png)

<br>

And then use the Group Mapping to import groups that we'd like to allow to authenticate on the portal

![x](/static/2025-08-27-captive-portal/12.png)

<br>

Then create a new Authentication Profile for LDAP Users

![x](/static/2025-08-27-captive-portal/13.png)

> Login Attribute field defines which LDAP attribute the firewall will use to match a username entered during login, userPrincipalName (UPN) is the Active Directory logon name in the format user@domain.com

<br>

Then we define which AD Groups are allowed to login based on the ones we imported earlier

![x](/static/2025-08-27-captive-portal/14.png)

<br>

At this point we can run a CLI test to make sure the LDAP Authentication Configuration is working

```
test authentication authentication-profile LDAP username helena password
```

![x](/static/2025-08-27-captive-portal/15.png)

<br>

Lastly we need to change the Authentication Profile on the Authentication Portal configuration

![x](/static/2025-08-27-captive-portal/16.png)

<br>

And now the portal will accept LDAP Users 

![x](/static/2025-08-27-captive-portal/17.png)

![x](/static/2025-08-27-captive-portal/18.png)

<br>







