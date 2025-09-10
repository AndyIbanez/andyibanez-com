---
title: "Quick Tip: Notifying Users of App Updates - For Free"
date: 2021-05-19T07:00:00-04:00
originalDate: 2021-05-17T09:16:42-04:00
publishDate: 2021-05-19T07:00:00-04:00
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
categories:
 - development
description: "Learn how to keep more of your users updated with free hosting and app version checking."
keywords:
 - swift
 - programming
 - apple
 - ios
 - ipados
---

This may  sound surprising to you, but even though we have app autoupdate on iOS now (and we have had it for a very long time), many people don't have it on, or the system simply doesn't prioritize app updates because users don't prioritize it enough. In fact, in my day job, in which I maintain a user-facing banking app, the vast majority of users are not even in the latest version. The most used version is the one we released in April, and we average one release per week for bug fixes alone, and about monthly for major new features.

We are bank and we really have no reason to do this for free, but my intention with the last paragraph was to tell you that your users may not be updating as often as you believe. If you login to App Store Connect and check your analytics, you may be surprised by how many people are not in your latest version. So in this article, I will show you how you can push users to update more often, and how you can do it for free.

# The Main Idea

The way to achieve is to simply store, somewhere, the latest app version that you have released to the public, and to have your app check against that version every so often, maybe on every lunch, or every 24 hours, the criteria is up to you.

My particular approach involves [hosting a JSON File on GitHub, which you can do for free](https://www.andyibanez.com/posts/quick-tip-hosting-json-files-github-for-free/). As a simple example, your JSON can look like this:

```
{
	"latestVersion": "5.2.9"
}
```

You have two choices here: You can either store your build number there, which would make it very simply to check against the build number inside your bundle, or you can store the short (aka "marketing") version of your app here, which requires a bit more work to parse the value.

Storing your build number may sound like a good idea but it depends on how you use build numbers in general. For my own apps, I always bump the build number for every version I upload to the App Store. For my jobby-job app, the build number gets reset for every new version. We only bump the build number when we need to modify the app before App Review and we need to do last minute changes, or when we deal with rejection and need to upload a new build.

In general, I prefer it when the build number keeps going up, so the approach I will show you here will use the Short Version.

## Short Version Parser

When you want to prompt users to update your app, you may need to consider if you want to simply let them know there is an ew version, or you may want to force them to update the app before using it. You may need to keep both situations in mind. For my jobby-job, we never force users to update unless we find critical bugs that affect the functionality of our apps. For new features, we won't force updates, as we try to keep older iOS versions in check.

A good strategy to have is you may want to force users to update when a new major version is available, and optionally prompt them when the minor version is bumped. You need to keep this in mind as you may just use the JSON file without the version altogether to set a flag and force users to update to any version any way.

To parse the version number and start doing checks, I have a `AppVersion` while which takes an app version as a string, and then you can compare it to other `AppVersion` objects or you can compare just each component - the major version, the minor version, or the patch version. The class looks like this:

```swift
class AppVersion {
    let appShortVersionString: String
    let appBundleVersionString: String
    
    public var major: Int {
        if let major = appShortVersionString.split(separator: ".").first {
            return Int(major) ?? 0
        }
        return 0
    }
    
    public var minor: Int {
        let splat = appShortVersionString.split(separator: ".")
        if splat.count > 1 {
            return Int(splat[1]) ?? 0
        }
        return 0
    }
    
    public var patch: Int {
        let versions = appShortVersionString.split(separator: ".")
        if let patch = versions.last, versions.count > 2 {
            return Int(patch) ?? 0
        }
        return 0
    }
    
    public var buildNumber: Int {
        return Int(appBundleVersionString) ?? 0
    }
    
    init(shortVersionString: String, bundleVersionString: String) {
        appShortVersionString = shortVersionString
        appBundleVersionString = bundleVersionString
    }
    
    static let shared = AppVersion(
        shortVersionString: Bundle.main.object(forInfoDictionaryKey: "CFBundleShortVersionString") as? String ?? "",
        bundleVersionString: Bundle.main.object(forInfoDictionaryKey: "CFBundleVersion") as? String ?? ""
    )
    
    var appShortVersion: String {
        appShortVersionString
    }
    
    var appBundleVersion: String {
        appBundleVersionString
    }
}

extension AppVersion: Equatable {
    static func == (lhs: AppVersion, rhs: AppVersion) -> Bool {
        lhs.major == rhs.major &&
        lhs.minor == lhs.minor &&
        lhs.patch == lhs.patch
    }
    
    static func === (lhs: AppVersion, rhs: AppVersion) -> Bool {
        lhs.major == rhs.major &&
        lhs.minor == rhs.minor &&
        lhs.patch == rhs.patch &&
        lhs.buildNumber == rhs.buildNumber
    }
}

extension AppVersion: Comparable {
    static func < (lhs: AppVersion, rhs: AppVersion) -> Bool {
        if lhs.major == rhs.major {
            if lhs.minor == rhs.minor {
                if lhs.patch == rhs.patch {
                    return false
                }
                return lhs.patch < rhs.patch
            }
            return lhs.minor < rhs.minor
        }
        return lhs.major < rhs.major
    }
}
```

A few things are worth pointing out:

* Easy access to the `major`, `minor`, and `patch` versions of the version.
* We conform to `Equatable` and add two comparing methods. `===` will compare the short and bundle version before deciding they are the same, and `==` only checks the short version number.
* By conforming to `Comparable`, we will be able to compare two versions to see if one is more recent or not - note that comparable only requires you to implement `<`, as the others can be synthesized for you.
* The `shared` property (which may make sense to rename to `current`) will give you the current app version.

With this, you now have a nice clean way to check for app versions, stored somewhere, and you can show UI and/or block functionality based on it.

# Conclusion

Users don't update their apps as often as we would like, so you may find it useful to let them know a new version is available without being invasive. Luckily, being able host simple data for free is really useful for this purpose, and version checking isn't that complicated either.