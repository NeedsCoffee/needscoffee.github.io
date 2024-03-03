---
id: 27
title: 'Decrypting a PFX file with OpenSSL'
date: '2008-01-10T11:36:20+00:00'
author: NeedsCoffee
layout: post
guid: 'https://myserverissick.com/?p=27'
permalink: /2008/01/decrypting-a-pfx-file-with-openssl/
categories:
    - Guides
    - 'IT Stuff'
tags:
    - certificates
    - openssl
    - 'pfx files'
---

Occasionally we have to host services that need securing with SSL and unfortunately we can only get our certificates in PKCS12 format, i.e. in a .pfx file – which is only suitable for installing in a Windows IIS box.  
So what do you do if you have to put a certificate that's in the form of a .pfx file into something that's asking for a private and a public key in plain text?! Well it's easy actually, we have to convert the .pfx file into something we can use. And thanks to the OpenSSL project there's a great and free tool for doing it.

1. Get and install OpenSSL from <https://slproweb.com/products/Win32OpenSSL.html>
2. Using the command line browse to where you installed openssl and then into the bin folder
3. Run this command, replacing \[path\] with the path to your certificate:  
    `openssl pkcs12 -in [path]\certificate.pfx -out [path]\certificate.pem -nodes`
4. You'll be asked for the password needed to decrypt the certificate at this point
5. Open your new certificate.pem in notepad and you'll see two sections, a private key section and a public key section. Those are what we're looking for. You can now copy and paste those into your service that's asking for the certificate.

By the way, here's a super page with a handy list of common openssl commands, very useful for a mostly Windows person who forgets commands frequently 🙂 [A few frequently used SSL commands](https://shib.kuleuven.be/docs/ssl_commands.shtml)
