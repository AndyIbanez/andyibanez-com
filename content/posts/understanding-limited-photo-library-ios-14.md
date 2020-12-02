---
title: "Understanding the Limited Photo Library in iOS 14"
date: 2020-12-02T07:00:00-04:00
publishDate: 2020-12-02T07:00:00-04:00
draft: true
originalDate: 2020-11-29T16:40:07-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - wwdc2020
categories:
 - development
description: "Learn how to implement features to deal with Apple's new privacy-focused photo library features."
keywords:
 - swift
 - programming
 - apple
 - wwdc2020
---

This year, Apple introduced a new feature that gives users even more control over what photos may third party apps see when they see a Photo Picker. The system will first present an alert asking users if they want to give access to their photos at all, and they have the option to give access to all their photos, or only to the photos they choose.

This is great, but it has been a very confusing experience for both users and developers alike. In this article we will explore this new privacy-focused photo picker and how to make good use of it without compromising too much of either usability and privacy.

# The New Limited Photos Library

Up until know, if an app asked for permission to use photos, the app got access to all of them through the PhotoKit APIs. In iOS 14, the limited photo access feature presents a system prompt where users can choose which photos will be available, and to which apps. That is to say, they can choose to give access to certain photos to one app, and a completely different set to other apps, meanwhile giving full library access to apps they fully trust.

The new photo app will even apply to apps that weren't shipped with iOS 14 support out of the box. As this is a system feature, the prompt, being a system one, can show up on top of any app.

Users can manage their selection in a few ways.

The first one is to send them to Settings. In the Settings for your app, users can modify the photo access. Once again, they will have the option to give access to the whole library, or only a few photos. When the user chooses to select photos, a new button to select photo access will appear under the option selector. Below are some screenshots I took for the Twitter app.

![All Photos](/img/IMG_7206.PNG)

![Selected Photos](/img/IMG_7207.PNG)

Note that when a user limits photo access to your app, you still have access to all the information you had before at an individual photo level. The functionality only limits access to assets themselves, not their metadata.

For apps that haven't made any changes to their photo library handling code, the app can prompt the user, once per launch session, if they want to keep their library selection or or if they want to change it.

## The Motivation

As we said above, this feature exists to give users more control over the data they make available to third party data. In a [WWDC2020 session video](https://developer.apple.com/videos/play/wwdc2020/10641/), Apple said that users don't like to give access to their whole libraries, because they have been growing over time, and they want to feel they have more control over that data.

## The Alternatives

The prompts we saw above are displayed to users when the app wants to access assets directly. In general, there's very little need to ask access to entire photo libraries.

Apple has introduced the new `PHPicker` class in iOS 14. This class is a replacement for `UIImagePickerController` which has many more features. Thew new picker provides a system UI to let users search their libraries and select multiple photos and videos. The bast part is, since this is a system prompt, the user will never be prompted about giving access to their photos to your app. Unless you have a good reason to request access to user photos directly (specific cases include photo backup apps), you should adopt this picker instead, which has many improvements over the old `UIImagePickerController`. It makes it better for users to search for the photo they want in huge libraries.

Also, in the past, camera apps had to request full read/write access to the library to be able to take photos and save them. This didn't make sense, as most camera apps would only want write access. There are new workflows to support this kind of situations.

## Adopting Explicit Limited Library Support

### Authorization Status

If you ever thought Apple wouldn't add more enum values to their APIs that have an authorization status, you now know they can. When requesting photo access, the system will give you back the new `.limited` case of `PHAuthorizationStatus` when users choose to limit access. There is also a new enum called `PHAccessLevel`. This value can be add only or read/write access.

### UI Considerations

Earlier we said that apps that don't have code to work with the limited library, can only change their access through Settings or by having the app automatically prompt the user if they want to change their selection, once per launch session.

To make the experience better, you should add a button or other UI elements to let users trigger the system prompt. To do this, you can call the new `presentLimitedLibraryPicker(from:)` method of `PHPhotoLibrary`.

```swift
let library = PHPhotoLibrary.shared()
library.presentLimitedLibraryPicker(from: viewController)
```

The system prompt can be annoying. The good news is that we can suppress it by setting `PHPhotoLibraryPreventAutomaticLimitedAccessAlert` key in our `Info.plist` to yes. If you already deal with the new limited photo library, you don't need the system to alert your users, as you have code to show the picker as necessary.

# Conclusion

The new limited photo library exists for privacy reason. Before you even make use of these APIs, you should ask yourself if you really need to access user photos, or if the new `PHPhotoPicker` can do the work for you. If you must access the assets directly, adopt the new features and suppress the alert to make the photo picker experience smoother.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!