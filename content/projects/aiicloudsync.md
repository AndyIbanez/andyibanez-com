---
title: "AIiCloudSync"
date: 2019-12-31T17:27:58-04:00
draft: false
categories:
 - projects
 - open source
tags:
 - ios
 - ipados
 - open source
 - swift
showcomments: false
showpagemeta: false
---

[GitHub Repository](https://github.com/AndyIbanez/AIiCloudSync)

<hr>

# AIiCloudSync

AIiCloudSync is a simple Package written in Swift to synchronize specific UserDefaults with the iCloud Key Value store (`NSUbiquitousKeyValueStore`).

To use this package, create a single instance of `AIiCloudSync`, and keep a reference to it. Once you create it, you don't need to worry about it any longer. It will automatically sync changes between iCloud and your local UserDefaults through the lifetime of your application. You can optionally receive notifications when the iCloud Syncs change so you can react accordingly.

# Usage

Using AIiCloudSync is as "set-and-forget" as possible. You create instances for all the User Defaults suite names you want to sync. You can optionally pass in `nil` if you want to synchronize with the standard User Defaults.

Add a property to your app delegate or scene delegate with the following code:

```swift
let myCustomDefaultsSync = AIiCloudSync(prefix: "sync", suiteName: "mySuiteName") // For defaults with a suite name
let standardDefaultsSync = AIiCloudSync(prefix: "sync") // For standard user defaults.
```

AIiCloudSync will only sync the defaults that have the specified prefix.

## Notifications

AIiCloudSync can notify you when it receives changes from the iCloud key value store. To receive notifications, you can register to receive  `AIiCloudSync.didUpdateToLatest` notifications from the notification center. When you receive this notification, the dictionary may include a `userInfo` dictionary with additional information. The framework has a  `AIiCloudSync.UserInfoKey` class with the following static properties to access the user info keys:

* `suiteName`: This key will be included and will contain the suiteName of the user defaults that just changed. If you initialized a `AIiCloudSync` object with no `suiteName`, this key will not be included.

The following snippet shows how you can register for these notifications:

```
let ns = NotificationCenter.default

ns.addObserver(
forName: AIiCloudSync.didUpdateToLatest,
object: nil,
queue: nil) { (notification) in
    // Do something with the notification
    
    // If you expect the notification to have a user info dictionary:
    if let userInfo = notification.userInfo {
        if let suiteName = userInfo[AIiCloudSync.UserInfoKeys.suiteName] as? String {
            print("Defaults with suite \(suiteName) did change")
        }
    }
}
```

If you don't understand how `NSNotificationCenter` works, you can read a primer on it [here](https://www.andyibanez.com/posts/nsnotificationcenter/).

## Other Recommendations

Avoid having User Default keys that use a dash symbol (`-`) in their name. Internally, AIiCloudSync uses this symbol for some operations in the sync progress.

# Changelog

## 1.0.0

- Initial release.

# Credits and Thanks

This project is a pure Swift implementation of [MKiCloudSync](https://github.com/MugunthKumar/MKiCloudSync), an Objective-C dependency for syncing User Defaults with the iCloud Ubiquity Store, created by Mugunth Kumar. I created this project because the original code hasn't seen an update in over 5 years, and because my own needs demand I avoid Objective-C as much as possible.

Additionally, this version has one more additional feature which lets you specify the `UserDefaults` to sync with using a suite name. The existing implementation can only use the standard User Defaults. This also allows you to synchronize iCloud defaults with multiple different User Defaults with different suite names.
