---
title: "Making Default Web Browsers and E-mail Clients May Not Be Possible For All Devs."
date: 2020-06-23T22:42:31-04:00
draft: true
categories:
 - development
tags:
 - apple
 - wwdc2020
---

I hate that my first WWDC article is going to be as lazy as this, but I thought I'd write about it anyway so as to at least give an starting point to devs who may attempt this.

Please note the vast majority of my thoughts here are based on a very short response I received on the dev forums. I may be awfully wrong in the end. I am leaving a link to my original question in the dev forums so you can check it out, and reach your own conclusions.

I spent a good potion - if not all - of my free time today trying to figure out how someone would make an app that can be set as the default Web Browser or Default E-Mail on iOS.

After not finding any documentation on how would devs create such an app, I asked on the [https://developer.apple.com/forums/thread/650027 Apple Dev Forums].

The answer I got makes me think that not all web browsers and e-mail clients may obtain the privilege of being able to be set as the default app on iOS.

To be able to be set as a default app, an iOS app must:

- Get the `com.apple.developer.default-web-browser` "managed entitlement". By "managed entitlement", I understand that only Apple can give you that entitlement. It wouldn't be the first APIs hidden behind an entitlement that not everyone can get.
- You must adopt minimum functionality. This is fully expected and I wouldn't argue against it.
- If you want more guidance on these steps, you have to email "default-app-requests@apple.com".

It looks like developers who want to create apps that can be set as a default for web browsing and e-mail will have to get through some bureocracy yet.

I understand why they'd make such a system, but hopefully, this will loosen with time. Maybe in a major release or two, the docs for default apps will be available to everyone and we won't have to go through much bureocracy.