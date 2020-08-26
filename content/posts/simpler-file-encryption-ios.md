---
title: "Simpler File Encryption on iOS"
date: 2020-08-19T07:00:00-04:00
originalDate: 2020-08-17T14:22:06-04:00
draft: false
publishDate: 2020-08-19T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - encryption
 - cryptography
categories:
 - development
description: "Learn how to use the Data Protection APIs on iOS and iPadOS."
keywords:
 - swift
 - encryption
 - cryptography
 - ios
 - ipados
---

It's not news that iOS has a heavy a focus on privacy and security. Apple provides us with many tools to make encryption easy, like [CryptoKit](https://www.andyibanez.com/posts/common-cryptographic-operations-with-cryptokit/), a high-level Cryptography framework on iOS. When [CryptoKit is not enough](https://www.andyibanez.com/posts/cryptokit-not-enough/), we can leverage older, lower-level APIs to do more cryptographic operations or use cyphers not covered by CryptoKit. We can even make use of [the Secure Enclave](https://www.andyibanez.com/posts/cryptokit-secure-enclave/) to leverage hardware-level security to our apps.

This is all cool and dandy but did you know that you don't need to leverage any of the technologies above to secure data in your app? In this article we will provide a much simpler method to protect user data, without having to know the first thing about Cryptography at all, and without compromising security at all. If you know you need to protect data, you can consider this option before even considering directly dealing with cryptography at all.

# Data Protection on iOS

Data Protection is an iOS feature that is automatically enabled the moment a passcode is set on the device. As this is enabled automatically, you get a lot of encryption support for free on your apps. You do not need to do anything especial when reading and writing files to ensure they are protected by Data Protection. Instead, you just use all reading and writing APIs as you would normally do, and the system will take care of encryption on the fly for you. This process is automatic, and when available, they use hardware-accelerated features.

So just by having a passcode, your data is already pretty secure. But to what extent? Is all my user data automatically safe just because a passcode is set? Well, no. But it's pretty darn close.

You can actually specify the data protection level on a file by file basis. You can apply any of four different levels to your files, and each level defines the conditions under which files may be accessed:

* **No Protection**: The file will always be accessible whether there is a passcode set or not.
* **Complete Until First Authentication**: The file will not be accessible until the user authenticates for the first time (after a reboot, etc). After unlocking the device, the file becomes accessible at all times, whether the device is currently locked or not. This is the device protection level.
* **Complete Unless Open**: You will only be able to open the file while the device is unlocked, but once you have it open, you can continue accessing it even after the user locks the device. You can create files with this protection level whether the device is locked or unlocked.
* **Complete**: You will only be able to access this file when the device is unlocked. No questions asked.

Since `Complete Until First Authentication` is the default level, your files are actually quite out there. This is not necessarily a bad thing, but be aware you can make your data be accessed only while the device is unlocked. Consider your use case, and apply the right data protection level as you deem fit.

## Applying a Different Data Protection Level to Your Files

Applying a different file protection level to a file is as easy as calling `Data.write(to:options)` on a piece of `Data` you want to store. In the `options` parameter (of type `NSData.WritingOptions`), you can pass in the protection level you want to use:

* `.noFileProtection`
* `.completeFileProtectionUntilFirstUserAuthentication` (default)
* `.completeFileProtectionUnlessOpen`
* `.completeFileProtection`

A small sample code would look like this:

```swift
let fileURL = // ...
let data = // ...
let protectionLevel = Data.WritingOptions.completeFileProtection

do {
    try data.write(to: fileURL, options: protectionLevel)
}
catch {
   // Handle errors.
}
```

And that's it! Once you save your data to a file, it will apply the protection level you want, protecting your data to the level you specify.

## Changing the Protection Level of a File

You can change the protection level of a file any time. The API is a bit messy at the time of this writing - it's Objective-C, so you need to cast, and you can't use the same `Data.WritingOptions` to change it, making it less intuitive. Luckily it's not too complicated.

To change the protection level, you need to use `NSURL`'s `setResourceValue(forKey:)` method:

```swift
do {
   try (fileURL as NSURL).setResourceValue(
                  URLFileProtection.complete,
                  forKey: .fileProtectionKey)
}
catch {
   // Handle errors.
}
``` 

`URLFileProtection` allows you to specify `.none`, `.completeUntilFirstUserAuthentication`, `.completeUnlessOpen`, and `.complete`.

## Managing Protected File Access

Depending on the file level you specify, there is a chance you will try to access protected data without it being available. So you need to be responsible and handle opening and closing your files, especially when working with higher-level data protection settings.

For this reason, your App Delegate can implement ` applicationProtectedDataWillBecomeUnavailable(_:)` and `applicationProtectedDataDidBecomeAvailable(_:)`.

Implementing this methods also implies using the right protection data level for each file. Files that may be accessed in the background due to tasks such as location, background push notifications, and the like, should have a flexible enough level to be accessible at all times.

# Conclusion

Protecting data in your app doesn't have to be complicated. By leveraging this simple system you can protect your data without knowing the first word about Cryptography at all. Just be responsible and ensure you know what data protection level you need to assign to each file, and your data will be more secure without affecting performance or functionality in any way.

This article is based on Apple's [Encrypting Your App's Files](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/encrypting_your_app_s_files) article.

<br>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.