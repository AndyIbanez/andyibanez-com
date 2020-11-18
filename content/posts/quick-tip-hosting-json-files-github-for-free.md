---
title: "Quick Tip: Hosting JSON Files on Github for Free"
date: 2020-11-04T07:00:00-04:00
originalDate: 2020-10-26T10:08:15-04:00
publishDate: 2020-11-04T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - macos
 - tvos
 - watchos
categories:
 - development
description: "Hosting remote files for your iOS apps for free."
keywords:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - macos
 - tvos
 - watchos
---

There are times in which you may want to host small JSON (or other small types of files) somewhere because your app needs them. Maybe you want to configure feature flags, or maybe you want to host IAP identifiers somewhere so as to not hardcode them in your app. This last case is something I did recently.

The immediate thought will be get a cheap server somewhere - after all, using something like [Vultr](https://www.vultr.com/products/cloud-compute/#pricing) you can get cheap hosting for as low as $2.50 per month. But did you know Github allows you to publish static websites, and you can piggyback that on that to store remote "config" about your apps?

Before moving, be aware that remotely configuring your app or using feature flags may get your app rejected or even removed post-approval. If you use this technique for feature flags, please consider being transparent with the App Review team, so they don't think you are passing off un-reviewed functionality to end users.

With that out of the way, let's host a small file in Github for small remote settings.

You can create a new repository or use an existing one without it clashing with your actual code. I prefer to keep a separate repository for all the simple files I use across my apps.

In the repository you want to host the files at, click **Settings* and scroll down until you find **Github Pages**. Enable the feature there. You will need to select the branch and optionally a Jekyll theme. We don't need Jekyll, which is a static site generator. Just select the branch, and you are done.

When you save it, Github will give you the link where all the content will be reachable.

In my case, the URL is `http://www.andyibanez.com/fairesepages.github.io/` because I have a custom domain which is in another repo where I host this website. If you don't have a custom domain, worry not, it will give you a URL you can print for free.

Then you just need to create the files and commit them there.

This is the file I have to create the Shop screen of Silvianna:

https://www.andyibanez.com/fairesepages.github.io/silvianna-iap.json

I then use this data in my app to build this screen:

![Silvianna Shop](/img/silvianna_shop.PNG)

Feel to use this technique when you need to host mostly static files. You will be able to change the files, but since you cannot run server-side logic, don't expect to do anything complicated that would require logic or backend storage.

# Conclusion

Sometimes we need to host small files, but there's no justification in paying a monthly fee for it. If you need to host absolutely simple files, you can go down this route.

You can also do this to host the minimum pages Apple requires for your app, including your Privacy Policy and Support Page.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.