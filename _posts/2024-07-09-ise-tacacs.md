---
title: Cisco ISE TACACS Configurations (Fortigate, F5, Juniper, etc)
date: 2024-07-09 10:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE, TACACS, Fortigate, F5, Juniper]
---

<br>

Here's the configuration to use Cisco ISE as the TACACS+ Server for various devices :

<br>

## Cisco IOS Device

First on Work Centers >> Device Administration >> Device Admin Policy Set, create a policy set for Cisco IOS Devices, with 2 types of Authorization, Admin and Guest

![x](/static/2024-07-09-ise-tacacs/01.png)

<br>

On Policy Elements >> TACACS Profile, create 2 profiles for Admin and Guest

![x](/static/2024-07-09-ise-tacacs/02.png)

![x](/static/2024-07-09-ise-tacacs/03.png)

<br>

Then create Command Sets for Admin to permit all commands and Guest with limited set of command allowed

![x](/static/2024-07-09-ise-tacacs/04.png)

![x](/static/2024-07-09-ise-tacacs/05.png)

<br>

Next configure TACACS on the Cisco IOS device

```shell
aaa new-model

tacacs server ISE_VM
 address ipv4 198.18.134.103
 key helena
 exit

tacacs server ISE_VM2
 address ipv4 198.18.134.104
 key helena
 exit

aaa group server tacacs+ TACACS_SVR
  server name ISE_VM

aaa authentication login default group TACACS_SVR local
aaa authentication enable default group TACACS_SVR enable 
aaa accounting update newinfo
aaa accounting exec default start-stop group TACACS_SVR
aaa accounting commands 0 default start-stop group TACACS_SVR
aaa accounting commands 1 default start-stop group TACACS_SVR
aaa accounting commands 15 default start-stop group TACACS_SVR
aaa authorization config-commands
aaa authorization exec default group TACACS_SVR local if-authenticated
aaa authorization commands 0 default group TACACS_SVR local if-authenticated
aaa authorization commands 1 default group TACACS_SVR local if-authenticated 
aaa authorization commands 15 default group TACACS_SVR local if-authenticated 
```
 
<br>

Run "show tacacs" to verify the configuration

![x](/static/2024-07-09-ise-tacacs/06.png)

<br>

Now logging in with user iseadmin grants us full access

![x](/static/2024-07-09-ise-tacacs/06a.png)

<br>

And with user iseguest we're limited to only limited commands

![x](/static/2024-07-09-ise-tacacs/06b.png)

<br>

On Operation >> TACACS Live Logs, we can see logs for both users authenticating

![x](/static/2024-07-09-ise-tacacs/07.png)

<br>

## Cisco ASA

For Cisco ASA, we create a new Policy Sets containing the same Command Sets and Shell Profiles used on Cisco IOS

![x](/static/2024-07-09-ise-tacacs/08.png)

<br>

Next configure TACACS on the Cisco ASA

```shell
aaa-server ISE protocol tacacs+
 reactivation-mode timed
 max-failed-attempts 2

aaa-server ISE (Management) host 198.18.134.103
 key helena
aaa-server ISE (Management) host 198.18.134.104
 key helena


aaa accounting command ISE
aaa authentication ssh console ISE LOCAL
aaa authorization command ISE LOCAL
aaa authorization exec authentication-server auto-enable
```

<br>

Run this command to test the TACACS authentication

```shell
test aaa authentication ISE host 198.18.134.103 username iseguest password cisco123
```

<br>

Now if we log in with iseadmin, we should be granted full access

![x](/static/2024-07-09-ise-tacacs/08a.png)

<br>

But if with isegues, we can only access limited commands

![x](/static/2024-07-09-ise-tacacs/08b.png)

<br>

Live Logs also shows the auth for both users

![x](/static/2024-07-09-ise-tacacs/09.png)

<br>

## Cisco WLC

Next for Cisco WLC, we'll also create a new Policy Sets with 2 Shell Profiles Authorization

![x](/static/2024-07-09-ise-tacacs/38.png)

<br>

The WLC_Admin Shell Profile grants full WLC access

![x](/static/2024-07-09-ise-tacacs/39.png)

<br>

Whereas the WLC_Guest grants only monitoring

![x](/static/2024-07-09-ise-tacacs/40.png)

<br>

Next we configure the TACACS on WLC, on Security >> AAA >> TACACS+ >> Authentication, add the ISE servers

![x](/static/2024-07-09-ise-tacacs/41.png)

![x](/static/2024-07-09-ise-tacacs/42.png)

<br>

Optionally, also add the ISE servers as the Accounting servers

![x](/static/2024-07-09-ise-tacacs/42a.png)

<br>

Next configure the authentication order on Prioriy Order >> Management User

![x](/static/2024-07-09-ise-tacacs/43.png)

<br>

Now we can login with iseadmin and iseguest and we'll be granted with its respective permission

![x](/static/2024-07-09-ise-tacacs/44.png)

![x](/static/2024-07-09-ise-tacacs/45.png)

<br>

The Live Logs shows the logging history as expected

![x](/static/2024-07-09-ise-tacacs/46.png)

<br>

## F5

First we create a Policy Sets for F5 with 2 Shell Profiles

![x](/static/2024-07-09-ise-tacacs/10.png)

<br>

For F5_Admin, we add an attribute "F5-LTM-User-Role" with Value "1"

![x](/static/2024-07-09-ise-tacacs/11.png)

<br>

And for F5_Guest, the "F5-LTM-User-Role" has Value of "2"

![x](/static/2024-07-09-ise-tacacs/12.png)

<br>

Now on the F5, on System >> User >> Remote Role Groups, we create 2 groups with the mapped attribute to correspond to its respective role

![x](/static/2024-07-09-ise-tacacs/13.png)

<br>

And then on Authentication, we enable TACACS+ and add the ISE Servers

![x](/static/2024-07-09-ise-tacacs/14.png)

<br>

Now if login with iseadmin, we are granted with Administrator role

![x](/static/2024-07-09-ise-tacacs/15.png)

<br>

With iseguest, the role is Guest

![x](/static/2024-07-09-ise-tacacs/16.png)

<br>

On ISE Live Logs, we can see both users authenticating

![x](/static/2024-07-09-ise-tacacs/17.png)

<br>

## Fortigate

For Forti, we also create a Policy Sets with 2 Authorization Shell Profiles

![x](/static/2024-07-09-ise-tacacs/18.png)

<br>

The Forti_Admin shell contains attribute "admin_prof" with value of "super_admin"

![x](/static/2024-07-09-ise-tacacs/19.png)

<br>

Wheraas the Forti_Guest has "admin_prof" with value of "read_only"

![x](/static/2024-07-09-ise-tacacs/20.png)

<br>

On the Fortigate, match those attributes set on ISE into Admin Profiles here, notice that we also add a new profile named "noaccess" which will be the default profile for all tacacs user until its overriden by the ISE attribute configuration

![x](/static/2024-07-09-ise-tacacs/23.png)

<br>

Next add the TACACS Servers

```shell
config user tacacs+
    edit "TACACS-SERVER"
        set server "198.18.134.103"
        set secondary-server "198.18.134.104"
        set key "helena"
        set authen-type pap
        set authorization enable
next
```

![x](/static/2024-07-09-ise-tacacs/21.png)

<br>

Then create a User Groups with the TACACS-SERVERS as the member

![x](/static/2024-07-09-ise-tacacs/22.png)

<br>

Now we create a admin user to actually login to Fortigate, there's basically 2 ways to do this, the first one is by matching the exact ISE user on the Fortigate. This means every ISE user authenticating to Fortigate needs to also be created here, which is tideous

![x](/static/2024-07-09-ise-tacacs/24.png)

<br>

The 2nd way is by using wildcard match, where it basically lets all users authenticated on ISE to access the Fortigate

![x](/static/2024-07-09-ise-tacacs/25.png)

<br>

After that, we need to access the CLI and add this command to ensure that ISE is able to assign the correct profile and override the "noaccess" profile given by default

![x](/static/2024-07-09-ise-tacacs/26.png)

<br>

Now if we try loggin in with iseadmin, we are give full super_admin access

![x](/static/2024-07-09-ise-tacacs/27.png)

<br>

But with iseguest, we are only given read_only access

![x](/static/2024-07-09-ise-tacacs/28.png)

<br>

ISE Live Logs shows both users authenticating with its respective permission

![x](/static/2024-07-09-ise-tacacs/29.png)

<br>

## Juniper






















