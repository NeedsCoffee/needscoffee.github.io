---
id: 53
title: Find a computer's model using PowerShell
date: '2013-09-21T00:00:00+01:00'
author: NeedsCoffee
layout: post
guid: 'https://myserverissick.com/posts/2013/find-a-computers-model-using-powershell'
permalink: /2013/09/find-a-computers-model-using-powershell/
categories:
  - IT Stuff
  - Scripts
tags:
  - automation
  - powershell
  - scripts
  - windows
published: true
---

Interestingly this blog's most popular post is one where I demonstrate [how to find the serial number of a PC in a batch file](/2008/04/find-a-computers-model-using-the-command-line-or-in-a-batch-file/ "Find a computer's model using the command line or in a batch file") so you can write a script that does different things for different computer models.

I pretty much don't write batch code any more, instead using PowerShell as much as possible. So I thought I'd add a post explaining how to get your computer model in PowerShell as well. If you've any experience using PowerShell then you know this could be a very short post...

And the answer?

`(Get-WmiObject -Class:Win32_ComputerSystem).Model`

Extending that to a specific model to allow us to work on something specific we have many options to play with. My preferred method if I'm only dealing with one model would be to filter at the point of making the query, like so:

`Get-WmiObject -Class:Win32_ComputerSystem -Filter:"Model LIKE '%H77ITX%'" -ComputerName:localhost`

Notice I've filtered the query using the `LIKE` operator which in WMI queries requires the `%` character as the wildcard indicator, not `*`. I've also specified the `ComputerName` as `localhost` in the above commands. Usually WMI commands operate on network objects, even if you only have the one PC, and as a result can take a little time to respond. If you specify `localhost` it speeds up the command and ensures you only get a result from the PC where you run your script. Of course if you want to do this on remote PCs then you'd use the `ComputerName` to specify one or more remote PCs.

Anyway that command is only going to return the object if we are running the query on the right PC model. That's not too useful on its own so we need to make it return something like `True` or `False`, then we can work inside an `If` statement perhaps. Actually we don't need to bother! `If` handles a returned object as if that means `True`, and *no* returned object as meaning `False`.

For example:

```
If(Get-WmiObject -Class:Win32_ComputerSystem -Filter:"Model LIKE '%H78ITX%'" -ComputerName:localhost) {
    Write-Host "Found an H78ITX model"
} else {
    Write-Host "Model not found"
}
```

OK so what do we do if we've got different models and we want to do different things on different models? Easy, go back to the first command, and use its output with the `Switch` statement.

```
$pcModel = (Get-WmiObject -Class:Win32_ComputerSystem).Model
Switch -wildcard ($pcModel) {
    "*Latitude*" { # install special apps for a Dell }
    "*Elite*"    { # install special apps for an HP }
    default      { # install other things for everything else }
}
```

There's plenty you can do with the switch statement, even using regex. There's a good explanation of it [here](http://technet.microsoft.com/en-us/library/ff730937.aspx "TechNet - Using the Switch Statement").

Of course you might want to do things for specific models, and also things for all models from a particular vendor. We can achieve that by being less specific with our initial command and just select different properties of the object from the variable like this.

```
$computerSystem = (Get-WmiObject -Class:Win32_ComputerSystem)
Write-Host $computerSystem<strong>.Manufacturer
Write-Host $computerSystem<strong>.Model
```

Hopefully that shows you how to get the model *and* manufacturer, then you can construct some logic around both.

Finally, something else to consider is that if you want to drill down using the serial number of a machine we need to make use of a different WMI object, namely Win32_BIOS, like this:

`(Get-WmiObject -Class:Win32_BIOS).SerialNumber`
