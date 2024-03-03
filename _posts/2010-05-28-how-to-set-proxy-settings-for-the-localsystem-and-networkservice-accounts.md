---
id: 37
title: 'How to set proxy settings for the LocalSystem and NetworkService accounts'
date: '2010-05-28T00:00:00+01:00'
author: NeedsCoffee
layout: post
guid: 'https://myserverissick.com/posts/2010/how-to-set-proxy-settings-for-the-localsystem-and-networkservice-accounts'
permalink: /2010/05/how-to-set-proxy-settings-for-the-localsystem-and-networkservice-accounts/
categories:
    - Guides
    - 'IT Stuff'
tags:
    - 'proxy servers'
    - 'system account'
---

Sometimes you need to set the proxy settings for the System account or NetworkService account on a server. There's a super easy way to do this using bitsadmin.

Set accounts to use a static proxy server with exclusions:

```
bitsadmin /util /setieproxy localsystem MANUAL_PROXY proxysrv:8080 ";*.consoto.com"
bitsadmin /util /setieproxy networkservice MANUAL_PROXY proxysrv:8080 ";*.consoto.com"
bitsadmin /util /setieproxy localservice MANUAL_PROXY proxysrv:8080 ";*.consoto.com"
```

Set accounts to use proxy.pac file:

```
bitsadmin /util /setieproxy localsystem AUTOSCRIPT http://contoso.com/proxy.pac
bitsadmin /util /setieproxy networkservice AUTOSCRIPT http://contoso.com/proxy.pac
bitsadmin /util /setieproxy localservice AUTOSCRIPT http://contoso.com/proxy.pac
```

Just replace the `contoso.com` and `proxysrv` bits with your own organisation's servers and pac file addresses. Then run the commands for the relevant accounts you want to change from an administrative command-line window.

More info can be found here: [https://msdn.microsoft.com/en-us/library/aa362813(VS.85).aspx](https://msdn.microsoft.com/en-us/library/aa362813%28VS.85%29.aspx)