---
id: 256
title: How to make Azure AD Connect disable expired accounts
date: '2019-01-10T13:45:06+00:00'
author: NeedsCoffee
excerpt: AADC doesn't process AD account expiry at all - here's a guide explaining how to make it do it!
layout: post
guid: 'https://myserverissick.com/?p=256'
permalink: /2019/01/how-to-make-azure-ad-connect-disable-expired-accounts/
categories:
  - Guides
  - IT Stuff
  - Security
tags:
  - aadc
  - account expiry
  - accountExpires
  - active directory
  - adsync
  - azure ad connect
  - dirsync
  - expiration
  - office 365
published: true
---

By default Azure AD Connect (AADC) does not honour account expiry in AD. So if you set an expiry date for an AD user thinking that will stop them accessing your synced Office 365 tenancy past a certain date, you'd be wrong! Why Microsoft haven't implemented this I really don't know, but it's easy to resolve, and important if you ask me. Here's how I do it...

First some background on the accountExpires attribute. This attribute is not a datetime attribute, it's another of the annoying AD attributes that have to be converted. Thankfully the later versions of AADC now have lots of handy functions to allow the creation of some complex sync rules, so this is actually possible now. Make sure you're on the latest version of AADC if you can. As of updating this article it was up to [v1.6.4.0](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/reference-connect-version-history).

When an account has NOT expired the attribute can actually take one of *three* different values:  
```
Null (no value)
0
9223372036854775807
```

When the account is going to expire the attribute value will be a value in the future, and when it has expired it'll be now or in the past, of course.

So we already have some rather odd behaviour. There's no single "never expires" value, and two of the values that do indicate no expiry are also possible candidates for expiry! We need to be mindful of these and filter for them when we configure our new sync rule.

### Concepts:
- We'll be creating a new user-\>person join rule on the AD connector that acts only on accounts that have an accountExpires value of relevance, and also only on accounts that have not already been disabled – we don't need to worry about those because the enabled/disabled state is something AADC already honours.

- Our new rule needs to validate if the accountExpires value is in the past or not, and if it is set AccountEnabled to False. If the account has not expired yet then it needs to tell the sync engine to ignore the rule output so other rules can act on the property if the need to. We do that by providing a Null output in this instance.

Let's get on with the rule...

1. Open the Sync Rules Editor and add a new Inbound rule. Give it an appropriate title, and set the precedence to something smaller than 100 so that it is a higher priority than the built-in rules.  
    ![](/images/image.png)
2. Click next and create 4 clauses as below.
    - `accountExpires : ISNOTNULL` (ignore accounts without an expiry value)
    - `userAccountControl : ISBITNOTSET : 2` (ignore disabled accounts)
    - `accountExpires : GREATERTHAN : 0` (ignore non-expiring accounts)
    - `accountExpires : LESSTHAN : 9223372036854775807` (ignore non-expiring accounts)  
    ![New sync rule – scoping filter](/images/image-1.png)
3. Click next twice and add a transformation as below. (*make sure to read the update at the bottom of this article too*)  
    - `accountEnabled : IIF(([accountExpires])<NumFromDate(Now()),False,NULL)`
    ![New sync rule – add transformation](/images/image-2.png)
    Then we click Save.

    Now this is NOT going to work straight away, and that is because AADC does not store the accountExpires attribute from AD out of the box. So we need to tell it to start reading the attribute.

4. Open your Synchronization Service Manager, making sure that no syncs are underway. If they are just wait for them to finish.
5. We should stop our AD sync scheduler so that we have some control over what's happening. Launch PowerShell and run this command disable the scheduler: `Set-ADSyncScheduler -SyncCycleEnabled:$False`
6. Back in the sync service manager double-click your AD connector. Switch to the Attributes section of the connector's properties window, click on "Show All" at the top-right and check the box next to "accountExpires"
    ![AD connector – Adding accountExpires attribute](/images/image-3.png)
7. Click OK to close the properties.
8. Now we need to run a full import to get this new data into AADC. Select the AD connector and click on Run from the actions on the right.
    ![](/images/image-5.png)
9. Now select "**Full Import**" and click OK.  
    ![](/images/image-6.png)  
    AADC will now import all the accountExpires values for the AD accounts. Once this completes we need to do a Full Synchronization as well or the new rule we've created won't be applied to existing accounts.
10. Repeat step 8. and choose "**Full Synchronization**" from the list.
    In a large environment this step usually takes quite a while. It can appear that a lot of changes are being made, but that will be because a full synchronization re-applies all the rules to the whole connector space. It's likely that very few will actually change, if any – unless you have lots of expired accounts.

    Last but not least we need to export the changes and give control back to the scheduler.
11. Repeat step 8. again but this time choose the **Azure AD** connector instead, and when you click Run choose the "**Export**" option.
12. Wait for step 11 to complete, then go back to PowerShell and fire this command to re-enable the scheduler: `Set-ADSyncScheduler -SyncCycleEnabled:$True`

## Important Update: 2021-06-10

Unfortunately I forgot to update this article after discovering a step I missed. The transformation we set in the new inbound rule we create (step 3) causes our rule to become something called a "temporal" rule – one that's affected by *time*. There's an issue with temporal rules and that is that MIM does not evaluate them propertly during a delta sync. In fact it only works properly during a full sync job or when the expiry attribute actually changes. That's because MIM uses a change engine and therefore normally only works on changes to attributes. Anyway the full MIM product has a special job that processes these rules once a day – in fact in MIM we create temporal *sets* for things like this and that's what actually gets evaluated in the daily job. Anyway sets are a MIM thing not a AADC thing and I'm getting off topic...

Why does this matter? Well sadly because we've incorporated *time* into our rule the expiry of an account is not going to get noticed when the actual expiry datetime arrives and nothing will change on the account to disable the user – so all our effort has actually gone to waste.

What's the solution? Well actually the solution is to run a full sync every so often. Pity really but that's what I've got. There's probably something cool we could do with the awesome [Lithnet](https://lithnet.io/microsoft-identity-manager-synchronization-service) [MIIS PowerShell module](https://github.com/lithnet/miis-powershell) (thanks Ryan – you're awesome!) but for now my answer has been to run a monthly full sync. I started by doing that manually when I remembered to do it (!) and last year I *bothered* to script it.

To run a full sync from PowerShell you use this command: `Start-ADSyncSyncCycle -PolicyType:Initial`
