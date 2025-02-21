---
title: Cisco ISE TACACS Configurations (Fortigate, F5, Juniper, etc)
date: 2024-07-09 10:30:00 +0700
categories: [Security, Cisco Identity Service Engine (ISE)]
tags: [ISE, Fortigate, F5, Check Point, ASA, Palo Alto]
---

<br>

Here’s the configuration setup for using Cisco ISE as the TACACS+ server across multiple devices:

<br>

## Cisco IOS

First on Work Centers >> Device Administration >> Device Admin Policy Set, create a policy set for Cisco IOS Devices, with 2 types of Authorization, Admin and Guest

![x](/static/2024-07-09-ise-tacacs/01.png)

<br>

On Policy Elements >> TACACS Profile, create 2 profiles for Admin and Guest

![x](/static/2024-07-09-ise-tacacs/02.png)

![x](/static/2024-07-09-ise-tacacs/03.png)

<br>

Then create Command Sets for Admin to permit all commands and Guest with limited set of allowed command 

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
 server name ISE_VM2

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

Run "show aaa-server" to verify the tacacs configuration

```shell
ciscoasa# show aaa-server 
Server Group:    LOCAL
Server Protocol: Local database
Server Address:  None
Server port:     None
Server status:   ACTIVE, Last transaction at 02:02:33 UTC Tue Jul 9 2024
Number of pending requests      0
Average round trip time         0ms
Number of authentication requests   11
Number of authorization requests    4
Number of accounting requests       0
Number of retransmissions       0
Number of accepts           10
Number of rejects           5
Number of challenges            0
Number of bad authenticators        0
Number of timeouts          0
Number of unrecognized responses    0

Server Group:    ISE
Server Protocol: tacacs+
Server Address:  198.18.134.103
Server port:     49
Server status:   ACTIVE, Last transaction at 03:25:51 UTC Tue Jul 9 2024
Number of pending requests      1
Average round trip time         61ms
Number of authentication requests   20
Number of authorization requests    41
Number of accounting requests       10
Number of retransmissions       0
Number of accepts           45
Number of rejects           9
Number of challenges            0
Number of bad authenticators        0
Number of timeouts          17
Number of unrecognized responses    0

Server Group:    ISE
Server Protocol: tacacs+
Server Address:  198.18.134.104
Server port:     49
Server status:   ACTIVE, Last transaction at 02:07:11 UTC Tue Jul 9 2024
Number of pending requests      0
Average round trip time         33ms
Number of authentication requests   1
Number of authorization requests    8
Number of accounting requests       7
Number of retransmissions       0
Number of accepts           10
Number of rejects           2
Number of challenges            0
Number of bad authenticators        0
Number of timeouts          4
Number of unrecognized responses    0
```

<br>

Now if we log in with iseadmin, we should be granted full access

![x](/static/2024-07-09-ise-tacacs/08a.png)

<br>

But if with iseguest, we can only access limited commands

![x](/static/2024-07-09-ise-tacacs/08b.png)

<br>

Live Logs also shows the auth for both users

![x](/static/2024-07-09-ise-tacacs/09.png)

<br>
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
<br>

## Cisco ACI

For APIC, we create another Policy Sets with also 2 Shell Profiles

![x](/static/2024-07-09-ise-tacacs/59.png)

<br>

APIC_Admin contains custom attribute "cisco-av-pair=shell:domains=all/admin/" to give it full permission

![x](/static/2024-07-09-ise-tacacs/60.png)

<br>

Whereas APIC_guest contains custom attribute "cisco-av-pair=shell:domains=all//read-all" to only give read-only permission

![x](/static/2024-07-09-ise-tacacs/61.png)

<br>

On APIC, go to Admin >> AAA >> Authentication >> Login Domains, create new Login Domain for Tacacs+ containing ISE Servers

![x](/static/2024-07-09-ise-tacacs/62.png)

![x](/static/2024-07-09-ise-tacacs/63.png)

<br>

That should do it, logging in with iseadmin now grants full admin access

![x](/static/2024-07-09-ise-tacacs/64.png)

![x](/static/2024-07-09-ise-tacacs/65.png)

<br>

Logging in with iseguest only grants read-only access

![x](/static/2024-07-09-ise-tacacs/66.png)

![x](/static/2024-07-09-ise-tacacs/67.png)

<br>

ISE Live Logs shows these users authenticating to ISE

![x](/static/2024-07-09-ise-tacacs/68.png)

<br>
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

On the Fortigate, match those attributes set on ISE into Admin Profiles here, notice that we also add a new profile named "noaccess" which will be the default profile for all tacacs user until it is overriden by attribute pushed by ISE

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

Then create a User Group with the TACACS-SERVERS as the member

![x](/static/2024-07-09-ise-tacacs/22.png)

<br>

Now we create an admin user to actually login to Fortigate, there's basically 2 ways to do this, the first one is by matching the exact ISE user on the Fortigate. This means every ISE user authenticating to Fortigate needs to also be created here, which is tideous

![x](/static/2024-07-09-ise-tacacs/24.png)

<br>

The second way is by using wildcard match, where it basically lets all users authenticated on ISE to access the Fortigate

![x](/static/2024-07-09-ise-tacacs/25.png)

<br>

After that, we need to access the CLI and add this command to ensure that ISE is able to assign the correct profile by overriding the "noaccess" profile given by default

![x](/static/2024-07-09-ise-tacacs/26.png)

<br>

Now if we try loggin in with iseadmin, we are given full super_admin access

![x](/static/2024-07-09-ise-tacacs/27.png)

<br>

But with iseguest, we are only given read_only access

![x](/static/2024-07-09-ise-tacacs/28.png)

<br>

ISE Live Logs shows both users authenticating with its respective permission

![x](/static/2024-07-09-ise-tacacs/29.png)

<br>
<br>

## Juniper

Next for Juniper, as usual we'll create a Policy Sets

![x](/static/2024-07-09-ise-tacacs/30.png)

<br>

For shell profile Juniper_Admin, we add attribute "local-user-name" with value of "JUNOS-RW"

![x](/static/2024-07-09-ise-tacacs/31.png)

<br>

And for Juniper_Guest, the "local-user-name" has value of "JUNOS-RO"

![x](/static/2024-07-09-ise-tacacs/32.png)

<br>

On the Juniper Device, we create 2 login class permission, one is Read-only, the other is Read-write full permission

```shell
set system login class RO-CLASS permissions view
set system login class RO-CLASS permissions view-configuration

set system login class RW-CLASS permissions all
```

<br>

Now we match the user attribute set on ISE to the its respective class permission

```shell
set system login user JUNOS-RO uid 2100 class RO-CLASS
set system login user JUNOS-RW uid 2101 class RW-CLASS
```

![x](/static/2024-07-09-ise-tacacs/33.png)

<br>

Then add the ISE as the tacacs servers

```shell
set system tacplus-server 198.18.134.103 source-address 198.18.134.82
set system tacplus-server 198.18.134.103 port 49
set system tacplus-server 198.18.134.103 timeout 10
set system tacplus-server 198.18.134.103 secret helena
set system authentication-order tacplus authentication-order password

insert system tacplus-server 198.18.134.104 after 198.18.134.103 

set system tacplus-server 198.18.134.104 source-address 198.18.134.82
set system tacplus-server 198.18.134.104 port 49
set system tacplus-server 198.18.134.104 timeout 10
set system tacplus-server 198.18.134.104 secret helena
set system authentication-order tacplus authentication-order password
```

![x](/static/2024-07-09-ise-tacacs/34.png)

<br>

That should do it, logging in with iseadmin grants us the RW-CLASS permission

![x](/static/2024-07-09-ise-tacacs/35.png)

<br>

And loggin in with iseguest grants us RO-CLASS permission 

![x](/static/2024-07-09-ise-tacacs/36.png)

<br>

The ISE Live Logs shows the authentication made by these 2 users on Juniper

![x](/static/2024-07-09-ise-tacacs/37.png)

<br>
<br>

## Check Point

Here we create another Policy Sets for Check Point

![x](/static/2024-07-09-ise-tacacs/47.png)

<br>

Then we give each of the Shell Profile its appropriate privilege level

![x](/static/2024-07-09-ise-tacacs/48.png)

![x](/static/2024-07-09-ise-tacacs/49.png)

<br>

On the Checkpoint, on User Management >> Roles, create a new role named "TACP-0" with only "TACACS_Enable" feature. This will be the default access for any user authenticating through tacacs, where then this user will be able to escalate the privile to TACP-15 through "tacacs_enable" if their permission allows

![x](/static/2024-07-09-ise-tacacs/50.png)

<br>

Then create another role named "TACP-15" with all features and commands checked. This will be the full fledged super admin user that can access basically everything

![x](/static/2024-07-09-ise-tacacs/51.png)

<br>

Next on Authentication Servers, add the ISE Servers and enable TACACS+

![x](/static/2024-07-09-ise-tacacs/52.png)

<br>

Now if we login with iseadmin, we'll be given TACP-0 role by default, which we can escalate by clicking TACACS+ Enable

![x](/static/2024-07-09-ise-tacacs/53.png)

<br>

And after that we now have TACP-15 role

![x](/static/2024-07-09-ise-tacacs/54.png)

<br>

If we login with iseguest, we'll also be given TACP-0 role by default, but we don't have permission to escalate the privilege to TACP-15

![x](/static/2024-07-09-ise-tacacs/55.png)

<br>

The same goes if we connect to CLI through SSH

![x](/static/2024-07-09-ise-tacacs/56.png)

![x](/static/2024-07-09-ise-tacacs/57.png)

<br>

And all the authentication logs can be viewed on Cisco ISE Tacacs Live Logs

![x](/static/2024-07-09-ise-tacacs/58.png)

<br>
<br>

## Palo Alto

On ISE, create a new Policy Set to handle traffic from Palo Devices

![x](/static/2024-07-09-ise-tacacs/74.png)

<br>

We also create 2 Tacacs Profiles with an attribute "PaloAlto-Admin-Role" and value of either "Read-Write" or "Read-Only", following the value of created Admin Roles on Palo Alto

![x](/static/2024-07-09-ise-tacacs/75.png)

![x](/static/2024-07-09-ise-tacacs/76.png)

<br>

On the Palo Device, create two Admin Roles for Read-Write and Read-Only roles

![x](/static/2024-07-09-ise-tacacs/71.png)

![x](/static/2024-07-09-ise-tacacs/72.png)

<br>

Then on Device >> TACACS+, create a new profile containing the ISE servers

![x](/static/2024-07-09-ise-tacacs/69.png)

<br>

Then on Authentication Profile, create a new Profile with type of "TACACS+" and the Server Profile created just now

![x](/static/2024-07-09-ise-tacacs/70.png)

<br>

Finally on Device >> Setup >> Authentication Settings, select the Authentication Profile

![x](/static/2024-07-09-ise-tacacs/73.png)

<br>

Now we can login to the Palo Device using the "iseadmin" with "Read-Write" role

![x](/static/2024-07-09-ise-tacacs/77.png)

<br>

And using the "iseguest" with "Read-Only" role

![x](/static/2024-07-09-ise-tacacs/78.png)

<br>

On ISE we can see the logs for these authenticating users

![x](/static/2024-07-09-ise-tacacs/79.png)

<br>








