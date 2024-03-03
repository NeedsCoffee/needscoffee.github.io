---
id: 288
title: Invalid Namespace with Start-ManagementAgent from LithnetMIISAutomation on a MIM Server
date: '2019-10-22T10:44:42+01:00'
author: NeedsCoffee
layout: post
guid: 'https://myserverissick.com/?p=288'
permalink: /2019/10/invalid-namespace-with-start-managementagent-from-lithnetmiisautomation-on-a-mim-server/
categories:
  - Uncategorized
published: true
---

I like to use the great PowerShell tools provided by [Lithnet](https://lithnet.io/microsoft-identity-manager-synchronization-service) on my MIM instances. Yesterday I discovered I couldn't start the management agents using Start-ManagementAgent and all I got back was "Invalid Namespace". After a bit of digging it turned out to be that the MIM WMI namespace had unregistered itself somehow.

Fixing that was quite simple but needed a slight adaptation of the advice from here: [http://www.apollojack.com/2009/04/cannot-connect-to-mms-wmi-service.html](http://www.apollojack.com/2009/04/cannot-connect-to-mms-wmi-service.html)

Solution:
```
regsvr32 "C:\Program Files\Microsoft Forefront Identity Manager\2010\Synchronization Service\Bin\mmswmi.dll"
mofcomp -N:root\MicrosoftIdentityIntegrationServer "C:\Program Files\Microsoft Forefront Identity Manager\2010\Synchronization Service\Bin\mmswmi.mof"
```
