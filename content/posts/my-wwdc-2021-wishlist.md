---
title: "My WWDC 2021 Wishlist"
date: 2021-05-26T07:00:07-04:00
originalDate: 2021-05-23T16:35:07-04:00
publishDate: 2021-05-26T07:00:07-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - apple
 - wwdc2021
 - wwdc
categories:
 - development
description:
 - My personal wishlist for WWDC 2021.
keywords:
 - apple
 - wwdc2021
 - wwdc
---

WWDC is undoubtly my favorite event of the year, and it has been for the past 11 years. This week I'm taking a break from writing technical articles and I want to talk about my personal wishlist, both for features and developers tools.

## Writing Actual ViewControllers for the Settings App

The iOS Settings.app was supposed to be a place where you as a user, have a centralized place to configure all your settings for all your apps. I have always liked the idea of having this central area for settings instead of having to have a dedicated settings screen within each app.

Unfortunately, as it has been the case for many years now, there is not really much you can do when you attempt to integrate with the settings app as it is now. You can create a Settings bundle, but it is all managed by a plist. You cannot have any more complex settings that would allow users to login to dedicated service accounts or do anything else remotely complex.

Back when I played around writing jailbreak tweaks, one of my favorite things to do was to write an entire Settings screen from scratch. You could add a view controller and therefore manage complex settings there. I used this in [Sideswitch Toggles](https://www.andyibanez.com/projects/sideswitch-toggles/) to dynamically load bundles in a Settings screen and in [Cecrecy](https://www.andyibanez.com/projects/cecrecy/) to load all user apps.

Having such feature would help me achieve my dream of having a centralized settings place for settings.

## Dedicated Debugging Tools for SwiftUI

SwiftUI is my favorite framework introduced in the past few years, but when it comes to debugging issues with it, it can sometimes be more complicated than I'd like. From performance issues to broken animations and unexpected view refreshes (or expected view refreshes that don't take place), sometimes SwiftUI throws you off the curve. While the framework is intuitive enough and you understand that views get rebuilt on state change, sometimes tracking down a state change is not easy as you could swear upon your family that you are not changing a given variable.

## More UIKit Components Represented in SwiftUI.

WWDC2020 got us many nice improvements over SwiftUI. Lazy grids and lazy stacks, pinned views in Scroll Views, the new SwiftUI lifecycle. Everything is beautiful, but the framework is still missing a lot of common and native functionality that users would expect. We cannot (easily) do pull to refresh, we cannot badge items in tab bar items, there is no search bar that support Search Tokens (introduced in WWDC2019)... There's a lot of stuff in UIKit that doesn't exist in SwiftUI yet. We can create a lot of it, arguably without much effort, but SwiftUI should have as much UIKit functionality as possible.

## Homescreen "Mini Apps"

Now hear me out on this one - When Apple introduced the first version of Widgets back in iOS 8, they said widgets are not "Mini Apps" and shouldn't be used as such. Despite that, some helpful widgets were exactly mini apps and they worked fine and where a beautiful convenience to have at times. From calculators to tip calculators, it was great being able to do some quick action without having to open a full app.

Fast forward to iOS 14, and Apple introduced a new widget system. While I love these widgets and actually use them constantly, they are mostly info widgets and you can't do much with them. Shall a widget be able to perform actions, they will launch the app. You cannot do much with them.

With the advent of M1 iPads, Apple has to do an amazing effort to convince us that putting a desktop-grade CPU in a tablet was worth every bit of the effort. For the first time ever, Apple tells us how much RAM is in each iPad, with the bigger storage models having way more RAM. This RAM has to be exploited somehow, and while we could argue it's all reserved for "pro" apps that don't exist yet, I believe any average consumer could make use of it, and what better way to do that than to have a bunch of mini apps on the iPad's homescreen.

## Widgets on the iOS/iPadOS Homescreens

If there's one thing I'd love to have is to put my favorite widgets in the lockscreen. While Face ID is crazy cast, it's still not better than simply having the info I care about at a glance on the lockscreen. I have wanted this for a long time, and I'd really like it to happen. The feature could support the privacy feature where notification contents are not visible until the device scans your face, similar to how Messages won't show anything about a message's notification without having to unlock the device, if you have such feature enabled.

## SwiftUI Support for Notifications Extensions

You can customize the look of your notifications UI by using Notification Content Extensions. You can add any UI you want and customize the user actions for them.

Currently, you need to use MVC for this, and while I think SwiftUI is perfect for this kind of tasks, it's just not supported yet.

## Being Able to Write Your own SwiftUI Result Wrappers

We saw our first taste of result wrappers - then called Property Wrappers - with the introduction of SwiftUI in 2019. While you can write your own property wrapper for anything, I haven't been able to find a way to write a property wrapper that works with SwiftUI in a way we can get the same behavior as `@State`, for instance. If you have a property wrapper to store keychain values and you update them, there's no way for SwiftUI to notice the change and rebuild your views as necessary.