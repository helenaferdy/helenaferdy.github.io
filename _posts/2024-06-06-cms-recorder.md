---
title: Cisco Meeting Server Recorder
date: 2024-06-06 17:30:00 +0700
categories: [Collaboration, Cisco Meeting Server (CMS)]
tags: [CMS, CUCM]
---

As seen on below's CMS components, we have installed the Core CMS, and to enable recording we will install addition CMS node but only for the Recorder component

![x](/static/2024-06-06-cms-recorder/00.png)

<br>

## Configuring NFS on Windows

But first let's prepare the NFS Share Path to store the recording, on the Server Manager create a Quick NFS Share

![x](/static/2024-06-06-cms-recorder/01.png)

<br>

Next set the location

![x](/static/2024-06-06-cms-recorder/02.png)

<br>

Set the name

![x](/static/2024-06-06-cms-recorder/03.png)

<br>

Then set the authentications to be "No server authentication"

![x](/static/2024-06-06-cms-recorder/04.png)

<br>

Then on Share Permission, we could make it only be accessible by CMS' ip address, but for simplicity we'll allow all machine to connect to this share

![x](/static/2024-06-06-cms-recorder/05.png)

<br>

Then on the folder permission, add Everyone and give it full control

![x](/static/2024-06-06-cms-recorder/06.png)

<br>

Review and create the NFS Share

![x](/static/2024-06-06-cms-recorder/07.png)

<br>

## Validating NFS Share

To validate the share, mount it on windows with this command "mount -o anon \\win1\recorder X:", this will mount the share on drive X.

![x](/static/2024-06-06-cms-recorder/08.png)

<br>
<br>

## Configuring CMS Recorder

Next we deploy another CMS node, this will be used solely for Recorder component

![x](/static/2024-06-06-cms-recorder/09.png)

<br>

Then give it password, set the ip address, dns, ntp, and then reboot the server

![x](/static/2024-06-06-cms-recorder/10.png)

<br>

Next using SFTP, uploads the signed certificate along with its key and the root ca

![x](/static/2024-06-06-cms-recorder/11.png)

<br>

Next run these commands to enable recorder

![x](/static/2024-06-06-cms-recorder/12.png)

```text
recorder sip listen a 5060 5061
recorder sip certs cms-rec.key cms-rec.cer helena-ca.cer
recorder nfs 198.18.134.21:/recorder
recorder enable
```

<br>

Now on the CMS Call Bridge Node, go to Configuration >> API >> /api/v1/callProfiles, create a new one

![x](/static/2024-06-06-cms-recorder/13.png)

> * recordingMode automatic sets this profile to automatically records a meeting whenever a participant joins.
> * sipRecordUri tells the callbridge where to send the recording to.

<br>

And on Configuration >> Outbound, we need to create the matching domain as configured on call profile that point to the CMS Recorder Node

![x](/static/2024-06-06-cms-recorder/14.png)

<br>

Next back on the API >> /api/v1/system/profiles, attach the callprofile here

![x](/static/2024-06-06-cms-recorder/15.png)

This will set all meetings and spaces to use this profile, which results in all meetings being recorded.

<br>

If we want a more specific configuration, we can just attach the callprofile to a specific space, like here for example on the mainspace 

![x](/static/2024-06-06-cms-recorder/16.png)

<br>
<br>

## Testing Meeting Recording

Now if we start a meeting, this will show a red dot on the left side indicating that the meeting is recorded. A voice prompt notifying the meeting is being recorded is also played

![x](/static/2024-06-06-cms-recorder/17.png)

<br>

On CMS Status >> Calls, we can see that an additional SIP Call Leg is created going out to CMS Recorder

![x](/static/2024-06-06-cms-recorder/20.png)

<br>

And if we look at the syslog messages on CMS Recorder Node, we can see the logs of the recording being initiated

![x](/static/2024-06-06-cms-recorder/21.png)

<br>

After the meeting ends, the recording will be saved as an mp4 file on the defined NFS path inside the folder of the its space id

![x](/static/2024-06-06-cms-recorder/18.png)

<br>

And the mp4 file can be played with any media player

![x](/static/2024-06-06-cms-recorder/18.png)

<br>


























