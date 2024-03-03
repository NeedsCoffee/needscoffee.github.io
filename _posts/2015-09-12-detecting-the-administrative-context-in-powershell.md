---
id: 58
title: Detecting the Administrative Context in PowerShell
date: '2015-09-12T00:00:00+01:00'
author: NeedsCoffee
layout: post
guid: https://myserverissick.com/posts/2015/detecting-the-administrative-context-in-powershell
permalink: /2015/09/detecting-the-administrative-context-in-powershell/
categories:
  - IT Stuff
  - Scripts
tags:
  - powershell
  - tips
published: true
---

I was writing a PowerShell script and it needed to know whether or not it was running in the administrative context. It's a bit fiddly but here's a short bit of PowerShell that sets a boolean variable for it:

```
$currentIdentity = New-Object System.Security.Principal.WindowsPrincipal([System.Security.Principal.WindowsIdentity]::GetCurrent())
$adminContext = $currentIdentity.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
```

Run that and $adminContext will be True if the script is running as admin, and False if not.
