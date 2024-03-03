---
id: 54
title: File extension for Windows Language Packs
date: '2015-02-04T00:00:00+00:00'
author: NeedsCoffee
layout: post
guid: https://myserverissick.com/posts/2015/file-extension-for-windows-language-packs
permalink: /2015/02/file-extension-for-windows-language-packs/
categories:
  - Guides
  - IT Stuff
tags:
  - cab files
  - file extensions
  - languages
  - mlc files
  - windows 7
  - windows 8
published: true
---

For some reason I had a mental block and couldn't seem to remember this, but instead of using dism to install language packs on Windows 7 or above, like this...

`dism /online /add-package /packagepath:<path to language pack .cab file>`

...you can also simply rename the .cab language pack file to have the **.mlc** file extension and double-click it.

There's yet another way too, and that's to run the lpksetup.exe utility and point it at your .cab file I think. Haven't tried that though as I think the .mlc method is easier.
