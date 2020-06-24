---
title: "The secret entitlements behind Default Apps on iOS"
date: 2020-06-23T22:42:31-04:00
draft: false
categories:
 - development
tags:
 - apple
 - wwdc2020
---

I hate that my first WWDC article is going to be as lazy as this, but I thought I'd write about it anyway so as to at least give an starting point to devs who will want to try creating their own default web browser or e-mail client on iOS and iPadOS.

Please note the vast majority of my thoughts here are based on a very short response I received on the dev forums. I may be awfully wrong. I am leaving a link to my original question in the dev forums so you can check it out, and reach your own conclusions. The link can be viewed by anyone and it's not locked behind an Apple Developer membership.

I spent a good amount - if not all - of my free time today trying to figure out how someone would make an app that can be set as the default Web Browser or default E-Mail client on iOS. After not finding any documentation on how would devs create such an app, I asked on the [Apple Dev Forums](https://developer.apple.com/forums/thread/650027).

The answer I got makes me think that not all web browsers and e-mail clients may obtain the privilege of being used as default apps on iOS.

To be able to be set as a default default web browser, an iOS app must:

- Get the `com.apple.developer.default-web-browser` "managed entitlement". By "managed entitlement", I understand that only Apple can give you it. It wouldn't be the first API hidden behind an entitlement that not everyone can get.
- You must adopt minimum functionality. This is fully expected and I wouldn't argue against it as link handling would have to be done exactly the same way Safari does it.
- If you want more guidance on these steps, you have to email "default-app-requests@apple.com". I was told to tell them about my app (I have no Web Browser in the works, so I won't e-mail them just yet), so I'm thinking they will choose to give you the entitlement or not based on what you tell them.

It looks like developers who want to create apps that can be set as a default for web browsing and e-mail will have to get through some bureocracy as of now.

I understand why they'd make such a system, but hopefully, this will loosen with time. Maybe in a major release or two, the docs for default apps will be available to everyone and we won't have to go through this process.