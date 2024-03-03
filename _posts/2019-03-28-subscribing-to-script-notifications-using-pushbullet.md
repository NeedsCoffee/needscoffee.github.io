---
id: 281
title: Subscribing to script notifications using Pushbullet
date: '2019-03-28T17:19:51+00:00'
author: NeedsCoffee
layout: post
guid: 'https://myserverissick.com/?p=281'
permalink: /2019/03/subscribing-to-script-notifications-using-pushbullet/
categories:
  - Uncategorized
published: true
---

In my job I run a lot of scripts. My scripts send me lots of emails and sometimes I need to act on information quickly – and sometimes I even take a holiday and someone else needs to do that instead... wouldn't it be awesome to be able to "subscribe" to those script notifications if and when you need to see them? Well here's how I do it using Pushbullet!

The simple principal of this is that we're going to get PowerShell to send a message making use of the REST api that Pushbullet exposes. We construct a simple json packet that contains the information and the channel on which it should appear. Then you just have to subscribe your Pushbullet client to that channel to get notifications whenever you need.

1. If you haven't got one yet, create a Pushbullet account – that's easy, do it here: [https://www.pushbullet.com](https://www.pushbullet.com)
2. There are clients on the website for consuming your notifications of course, grab and login to one of those next.
3. Now get an API key so your script can send messages. Go here [https://www.pushbullet.com/#settings/account](https://www.pushbullet.com/#settings/account) and click the "Create Access Token" button, then take a copy of the text you get – it'll look like this: o.abcdefgh0123456789ijklmnopqrstuv
4. Finally create a channel where you want notifications to appear. Go to [https://www.pushbullet.com/my-channels](https://www.pushbullet.com/my-channels) and create a new channel. The bit you'll need for the function to work is the "tag" of the channel. You can see that in the url of the channel ([https://www.pushbullet.com/channel?tag=<this-bit-is-the-tag>](https://www.pushbullet.com/channel?tag=<this-bit-is-the-tag>))
5. OK now add two variables to your script, $pb_config_channel_tag and $pb_config_api_key. The first needs to be set to the channel tag you got in step 3, and the second to the access token string you got in step 2.
6. Add the function into the script then call it with the message you want to send!

Here is my function:

```
Function Send-PbChannelMsg ($title = (Get-Date).DateTime, $message = "", $channel = "$script:pb_config_channel_tag", $apikey = "$script:pb_config_api_key") {
    # Pushbullet API is defined here https://docs.pushbullet.com/#create-push
    $pb_config_api_uri = "https://api.pushbullet.com/v2/pushes"
    # make a new object with the fields needed for the message
    $pb_packet = [PSCustomObject]@{
        type = "note"
        title = "$title"
        body = "$message"
        channel_tag = "$channel"
    }
    # now test to make sure we have the minimum needed for a successful message
    $pb_ready = $true
    if($pb_packet.channel_tag.Length -eq 0){
        Write-Warning 'Channel tag missing. Please define $pb_config_channel_tag or pass to function using -channel switch'; $pb_ready = $false
    }
    if($apikey.Length -eq 0){
        Write-Warning 'Pushbullet API key missing. Please define $pb_config_api_key or pass to function using -apikey switch'; $pb_ready = $false
    }
    if($pb_ready){
        # setup the access-token header
        $pb_header = @{"Access-Token"=$apikey}
        # call the rest api and convert the message object into json
        Invoke-RestMethod -Uri:$pb_config_api_uri -Body:($pb_packet | ConvertTo-Json) -Method:Post -Headers:$pb_header -ContentType:"application/json"
    }
}
```

You can call the function like this:  
`Send-PbChannelMsg -title "Notification from $env:computername" -message "Disk space = $("{0:N2}" -f ((GWMI -Class Win32_Volume -Filter "DriveLetter = 'C:'").FreeSpace/1GB))" -channel "my-test-channel" -apikey "o.abcdefgh0123456789ijklmnopqrstuv"`

Then your notification will appear a bit like this.  
![](/images/image-7.png)

Just a few things to bear in mind:

- Pushbullet channels are public so if someone finds out your tag they could read your notifications – this might actually be wanted if you'd like to share notifications with other people, but try to keep your tags complex so they can't be discovered easily
- Make sure your notifications don't have any 'secret' data in them – these things are public after all!
- Don't abuse the API – if you were to trigger a notification for every occurrence of a particular event, and that event suddenly cropped up 1000 times you'll send 1000 push message! Might be worth putting in a delay in the code or some fail-safes to prevent spamming the API.
- You'll be sending a block of text in the notification – so make sure your message is sanitised.
