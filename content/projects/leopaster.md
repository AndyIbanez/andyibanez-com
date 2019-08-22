---
title: "Leopaster"
date: 2014-01-04T22:15:00-04:00
draft: false
description: "Code snippet sharing tool for Ghostbin"
showcomments: false
showpagemeta: false
categories:
 - projects
tags:
 - macos
 - open source
---

![Leopaster](/img/leopaster.png)

*Discontinued project. Developed in 2015*

Quick Status Bar Menu on OS X that allows you to quickly paste code on @DHowett’s ghostbin.com.

ghostbin.com is a simple pasteboard service that allows people to paste code or any other text and to quickly share a link to them. These services allow developers to share code snippets faster to either request help with an specific piece of code, or to help others with theirs (Other uses may apply).

Usage

Leopaster is a simple status bar app on your Mac. When you click it, you will see all it’s features and Ghostbin’s supported languages.

Copy the code you want to send to Ghostbin to your clipboard (aka right click -> copy, or cmd+c). Then click the Leopaster item and select an expiration time. By default, the expiration time is set to “Undefined”, which doesn’t send any expiration data to Ghostbin so it’s kept indefinitely. Changing the expiry time will use that expiry data on all further pastes (it’s not necessary to choose a different expiry time everytime you want to use Leopaster as it will remember your last used setting).

Simply clicking on the language will send your code in your clipboard to Ghostbin. When your paste is ready, you will receive a notification on OS X telling your your paste is ready, and it will place the link your clipboard, so you can paste it (aka right-click -> paste, or cmd+v) anywhere you want, hassle free.

Notes and Credits

Thanks to @DHowett for very quickly giving me a link to Ghostbin’s API usage. Finding the API URL is a bit complicated unless you ask, so I will put it here for those interested:

https://ghostbin.com/paste/p3qcy

And finally thanks to anyone who is planning on using this.

<hr>

Available source code (Warning - it's bad): [Github](https://github.com/AndyIbanez/Leopaster)