---
title: "Using the iOS Keychain with Biometrics"
date: 2020-06-03T07:00:00-04:00
originalDate: 2020-05-31T12:30:02-04:00
publishDate: 2020-06-03T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - macos
 - ipados
 - keychain
categories:
 - development
description: "Learn how to use the iOS Keychain with Biometrics, such as Touch ID or Face ID."
keywords:
 - swift
 - ios
 - ipados
 - keychain
---

If you have been [using the keychain on your iOS apps](https://www.andyibanez.com/posts/using-ios-keychain-swift/) you may want to start using Face ID/Touch ID to let your user access your app and their data. This is a common use case but it's very easy to do incorrectly.

Apple introduced Touch ID all the way back in 2013, and ever since then, every iOS device has come with some sort of biometric authentication method, be it Touch ID or Face ID. This has allowed developers to implement convenient unlocking into their apps to access sensitive data without having to ask for the passcode. If your app "locks" access in any way your users are probably expecting to "unlock" with their finger or Face ID, so it is your responsibility to implement in a way that is secure and can't be vulnered.

# The Wrong Way

If you have implemented Touch ID or Face ID in your app before, you have probably seen articles or tutorials that let you to grab the boolean of a biometric authentication operation and work from there. Something like this:

```swift
  func requestBiometricUnlock() {
    let context = LAContext()
    
    var error: NSError? = nil
    
    let canEvaluate = context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error)
    
    if canEvaluate {
      if context.biometryType != .none {
        print("We got a biometric")
        context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "To access your data") { (success, error) in
          if success {
            print("Authenticated successfully!")
          }
        }
      }
    }
  }
```

In the above code, first we check if we have any available biometrics. If we do, we will try authenticating with the existing biometric. Once the function calls our closure, it will contain a boolean indicating if the authentication succeeded or if it failed. If it was successfull, you can access your data.

This is actually a very naïve approach because in theory, a jailbreak tweak could hook into your app, trick it to believe a successful biometric scan was performed, and steal any information behind that simple `if true` call.

This code *probably* has some uses and it should be used strictly for convenience and not for actual security. The right way to lock sensitive data in the keychain while allowing for biometric access is to create your keychain item with a specific access policy, also known as an access control.

There is a nice article [here](https://medium.com/@pig.wig45/touch-id-authentication-bypass-on-evernote-and-dropbox-ios-apps-7985219767b2) about how easy it is to bypass the Touch ID/Face ID prompt, using the Evernote and Dropbox iOS apps as examples.

# Keychain and Access Control

When you create an access control, you specify two conditions under which a keychain item should be available:

* The level of accessibility for the item: Whether you want the item to be accessible after the user unlocks the device, every time they authenticate, or other.
* The authentication level: This flag lets us specify if the item can be accessed any time, as in, only when the user is present, and more. This condition lets us set the biometric conditions to allow our keychain items to be accessed.

These APIs are governed by the `Security` framework just like the keychain, so make sure you import it before moving on.

## Creating An Access Control.

To create an access control, you use the `SecAccessControlCreateWithFlags` function. This function takes four parameters, of which we mostly care about the second and third one.

```swift
let accessControl = SecAccessControlCreateWithFlags(
  nil,
  kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly,
  .userPresence,
  nil)
```

The first parameter is an allocator. When we use nil, we use the default one. I do not know when it would be appropriate to use a different allocator.

The second parameter allows us to specify the accessibility level. Using the `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly` constraints the item into only being accessed when the device has a passcode set (the `WhenPasscodeSet` part of the constant name) - if the device has no passcode, trying to add a new keychain item is not going to do anything -, and for this device only (the `ThisDeviceOnly` part of the constant), meaning the item will not be shared via iCloud Keychain or even from a backup. It is exclusive to the device that created it.

The `SecAccessControlCreateFlags.userPresence` flag lets us specify that we want the user to be there when the keychain is accessed. To do this, under this specific flag, the system will prompt for a biometric authentication, falling back to the device's passcode when necessary. You can restrict the item to use only biometric with `.biometricCurrentSet` or only the device passcode with `.devicePasscode`. You have a lot of flexibility, but `.userPresence` is the most used one, as it defaults to biometrics and fallbacks to passcode automatically.

## Adding Items With An Access Control.

To add a new keychain item with your new access control, you don't have to do much different like when you add an item with `SecItemAdd`. The only difference is your query will have a `kSecAttrAccessControl` key with your access control.

```swift
let accessControl = SecAccessControlCreateWithFlags(
  nil,
  kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly,
  .userPresence,
  nil)!

let query = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrAccount: "andyibanez",
  kSecValueData: "Pullip2020".data(using: .utf8)!,
  kSecAttrServer: "pullipstyle.com",
  kSecAttrAccessControl: accessControl,
  kSecReturnData: true
] as CFDictionary

var result: AnyObject?

let status = SecItemAdd(query, &result)
```

As you can see, our dictionary is not too different. One key to set the access control and that is all we need. From now on, whenever we need this item, we will need to authenticate the user, either with biometrics or passcode (because we used the `.userPresent` setting).

## Retrieving Items with Authentication

To retrieve items, once again you just have to write a common query. But you do not have to specify the access control this time.

```swift
let searchQuery = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrAccount: "andyibanez.com",
  kSecAttrServer: "pullipstyle.com",
  kSecMatchLimit: kSecMatchLimitOne,
  kSecReturnData: true,
  kSecReturnAttributes: true,
  kSecUseOperationPrompt: "Access your data"
] as CFDictionary

var item: AnyObject?

let status = SecItemCopyMatching(searchQuery, &item)
```

The system is smart enough to prompt for the biometrics when it finds an item that was created with a given access control. The `kSecUseOperationPrompt` key allows us to specify a user-visible string, and it's optional.

You may be wondering how does this work when your query matches multiple items, some of which may not have an access control associated to them? If you want to perform a wide search, the item will return all the items that match, but it will ask you to provide authentication for every protected item. You can perform wide searches skipping the ones with a biometric access control by providing the `kSecUseAuthenticationUI` key with the `kSecUseAuthenticationUISkip` value in your search query.

**Note:** To test this, you will need to run your code in a real device. Because you can't set a device passcode in the simulator, this code runs and finishes immediately after you run it, even if you enroll Touch ID/Face ID in the simulator. This gives you the impression it is not working properly, so keep that in mind if you are using this feature.

# Conclusion

Using biometrics with the keychain is very easy, so you should definitely use it if you are planning to hide credentials behind Touch ID/Face ID. I've seen a lot of code who don't do this, and it is dangerous to do. There's very little use to use the LocalAuthentication framework directly when dealing with credentials, so use the keychain this way when you need to.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.