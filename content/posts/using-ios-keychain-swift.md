---
title: "Using the iOS Keychain in Swift"
date: 2020-05-27T07:00:00-04:00
originalDate: 2020-05-24T12:35:43-04:00
publishDate: 2020-05-27T07:00:00-04:00
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
 - tvos
 - watchos
 - ipados
 - keychain
categories:
 - development
description: "Learn how to use the iOS Keychain in Swift."
keywords:
 - swift
 - ios
 - tvos
 - ipados
 - watchos
 - keychain
aliases:
 - /using-ios-keychain/
---

*This article is an entirely rewritten version of an old tutorial I wrote years ago titled "Using the iOS Keychain". Originally written in Objective-C, the old version has been archived but it is accessible [here](https://www.andyibanez.com/old-content/).*

The Keychain is the place where you would store sensitive data. As secure as iOS currently is, the keychain is the right place to store passwords, authentication tokens, and other sensitive data. You should not store this kind of data in `UserDefaults`, even if iOS has made it harder to access that data for normal users in the latest versions.

In this article we will explore how to use the iOS keychain (which is also applicable to iPadOS, watchOS, and even tvOS) using Swift. The APIs are similar to the ones used in macOS, but the way both systems work with their keychain is different enough to consider them separate. The keychain APIs are very old and as such we will be written some "ugly" Swift code to get everything to work, although these days it's much easier to do the bridging to Core Foundation and back. With that said, using the keychain isn't too hard, and you should use it if you find yourself needing to store sensitive data.

# Basic Keychain Concepts

Before we write some code, we need to get some terminology down. Keep these concepts in mind as you read through this article:

* **Keychain**: The keychain is a secure and encrypted storage place for sensitive data. You can think of it is a database of sensitive information.
* **Keychain Item**: This is a registry in the keychain.
* **Item Class**: You can think of a class as a template of information you want to store. The keychain offers classes for different common credentials, such as username/password pairs, a certificate, a generic password, and more.

Please note that the keychain is tied to the developer provisioning profile used to sign the app and its bundle ID. If either of these change, the data becomes inaccessible.

# Common Keychain Operations

With that basic terminology out of the way, we can start doing basic operations: Adding new items to the keychain, searching for specific items, updating items, and deleting items.

First things first, the keychain services are part of the `Security` framework, so don't forget to add that import:

```swift
# import Security
```

## Adding Items to the Keychain

### Adding Items

To add items to the keychain, you use the `SecItemAdd` function. Again, these APIs are very old, so you may be surprised by what they take as arguments and what they return. The first parameter is a dictionary known as a *query*. The query specifies the data the keychain item will hold along with parameters we can use to find it - more on that later. Being an old (and low level) API, this argument is actually a `CFDictionary`.

At the very least, the keychain item should have:

* The item class, specified with the `kSecClass` key.
* The actual data you want to hold, whether it is a plain password, or anything else. This should be stored as `Data` (or if you prefer, `CFData`). The key is `kSecValueData`.

You can specify optional attributes and optional return types as well. We will see these in a bit.

The second parameter to this function is a `UnsafeMutablePointer<CFTypeRef>?` which may contain the data created. This is an unsafe generic pointer. This is low level stuff, but I promise it will make sense later.

This parameter makes more sense when are retrieving items from the keychain, but if you wanted to return the item you just created to check its data or do anything else with it, you can use this parameter to return it after creating it.

The function itself returns an `OSStatus`. `OSStatus` tells us the status of the operation we just performed, and you will see it in all the keychain APIs. If it returns `0` (or `errSecSuccess`), the operation finished successfully and our operation finished without an issue. In the case of `SecItemAdd`, that will mean our new item was added to the keychain.

To create a keychain item, this is the minimum code you would need:

```swift
let keychainItemQuery = [
  kSecValueData: "Pullip2020".data(using: .utf8)!,
  kSecClass: kSecClassGenericPassword
] as CFDictionary

let status = SecItemAdd(keychainItemQuery, nil)
print("Operation finished with status: \(status)")
```

This will work fine, and it will print `0` for the status which is what we would expect (Yes, `OSStatus` is not bridged to Swift yet, so you won't get the actual enum case name `errSecSuccess`). The problem with this code is that searching for this item later will be a bit harder. We will explore why when we explore how to retrieve items, but for now, try to add more context to your new items. We can do this by adding optional attributes. All the attributes we can use are also dictionary keys that have the `kSecAttr` prefix.

Different classes have different attributes that make the keys unique. When using `kSecClassInternetPassword`, we can specify a username and the domain name of the website. Note that we are passing `nil` to the second argument, because we aren't interested in returning the newly created item just yet.

```swift
let keychainItem = [
  kSecValueData: "Pullip2020".data(using: .utf8)!,
  kSecAttrAccount: "andyibanez",
  kSecAttrServer: "pullipstyle.com",
  kSecClass: kSecClassInternetPassword
] as CFDictionary

let status = SecItemAdd(keychainItem, nil)
print("Operation finished with status: \(status)")
```

### Retrieving Newly-Added Records.

`SecItemAdd` has a second argument we can use to return the newly created item if we want to use it instantly. This parameter is called `result`, and you will probably specify `nil` here most of the time, but it's good to know that this function can return data after adding items, in case you ever need it.

When we create a new item, we can specify four different return types that will be filled in the second argument. Remember this parameter is of type `UnsafeMutablePointer<CFTypeRef>?`, so it can result a pointer that can point to pretty much anything. This is a big mess, so we will explain the different return types before we move on. All the return types are specified with a key in the query that has the prefix `kSecReturn`.

* `kSecReturnRef`: When this is set to `true`, `result` will point to either a `SecKeychainItem`, `SecKey`, `SecCertificate`, `SecIdentity`, or `CFData`, depending on the `kSecClass` specified in the query. I couldn't get this to return anything on iOS.
* `kSecReturnPersistentRef`: When this is set to `true`, `result` will contain `CFData` which you can use to persist on disk or pass to different processes.
* `kSecReturnData`: When this is set to `true`, this will return the actual sensitive data stored in the keychain item. The sensitive data will vary on the item class, but if your query contains a `kSecValueData` key, it will return that.
* `kSecReturnAttributes`: This will return all the attributes used to create the item in a `CFDictionary`.

So... Yes, it is a mess. The API is very old, and `result` can be pretty much anything. This API is not really friendly with Swift's type safety, so we have to do a lot of casting when working with keychain services.

`CFTypeRef` is bridged to `AnyObject`, so you can forget about `CFTypeRef` and use `AnyObject` everywhere instead.

Here is an example of a query that returns the attributes, using `kSecReturnAttributes`:

```swift
let keychainItem = [
  kSecValueData: "Pullip2020".data(using: .utf8)!,
  kSecAttrAccount: "andyibanez",
  kSecAttrServer: "pullipstyle.com",
  kSecClass: kSecClassInternetPassword,
  kSecReturnAttributes: true
] as CFDictionary

var ref: AnyObject?

let status = SecItemAdd(keychainItem, &ref)
let result = ref as! NSDictionary
print("Operation finished with status: \(status)")
print("Returned attributes:")
result.forEach { key, value in
  print("\(key): \(value)")
}
```

This will print:

```
Operation finished with status: 0
Returned attributes:
acct: andyibanez
atyp: 
sha1: {length = 20, bytes = 0x589a101265fbb5cd7b596657d2109c13450533a1}
path: 
sdmn: 
pdmn: ak
mdat: 2020-05-24 19:33:51 +0000
sync: 0
cdat: 2020-05-24 19:33:51 +0000
ptcl: 0
srvr: pullipstyle.com
agrp: 7X7FABXK4C.com.andyibanez.keychain
port: 0
```

You will never use these keys directly. Always use the proper `kSecAttr` key to get your data:

```swift
print("Website: \(result[kSecAttrServer] ?? "")") // Website: pullipstyle.com
```

You may have noticed the actual password is missing. To get the password, we need to specify the `kSecReturnData` key instead. `kSecReturnData` returns a `CFData`, and `kSecReturnAttributes` returns a dictionary, so they are incompatible types to begin with.

To retrieve the password itself:

```swift
let keychainItem = [
  kSecValueData: "Pullip2020".data(using: .utf8)!,
  kSecAttrAccount: "andyibanez",
  kSecAttrServer: "pullipstyle.com",
  kSecClass: kSecClassInternetPassword,
  kSecReturnData: true
] as CFDictionary

var ref: AnyObject?

let status = SecItemAdd(keychainItem, &ref)
let result = ref as! Data
print("Operation finished with status: \(status)")
let password = String(data: result, encoding: .utf8)!
print("Password: \(password)")
```
```
Password: Pullip2020
```

... But, you can actually return both at the same time! Even though using `kSecReturnData` and `kSecReturnAttributes` return different types, if you specify both keys at the same time and set them to true, `SecItemAdd` will return a `CFDictionary` that contains the attributes *and* the password. So, if you are aiming for consistency, and you always want `result` to have the same data type, if you specify more than one return key, you will always get a dictionary back.

```
let keychainItem = [
  kSecValueData: "Pullip2020".data(using: .utf8)!,
  kSecAttrAccount: "andyibanez",
  kSecAttrServer: "pullipstyle.com",
  kSecClass: kSecClassInternetPassword,
  kSecReturnData: true,
  kSecReturnAttributes: true
] as CFDictionary

var ref: AnyObject?

let status = SecItemAdd(keychainItem, &ref)
let result = ref as! NSDictionary
print("Operation finished with status: \(status)")
print("Username: \(result[kSecAttrAccount] ?? "")")
let passwordData = result[kSecValueData] as! Data
let passwordString = String(data: passwordData, encoding: .utf8)
print("Password: \(passwordString ?? "")")
```

```
Operation finished with status: 0
Username: andyibanez
Password: Pullip2020
```

## Retrieving Items from the Keychain

In this section we will explore how to write queries that retrieve data from the keychain. If you want to work along, I have written this small piece of code that will populate the keychain with different but similar entries:

```swift
let usernames = ["andyibanez", "alice", "eileen", "blackberry"]

usernames.forEach { username in
  let keychainItem = [
    kSecValueData: "\(username)-Pullip2020".data(using: .utf8)!,
    kSecAttrAccount: username,
    kSecAttrServer: "pullipstyle.com",
    kSecClass: kSecClassInternetPassword,
    kSecReturnData: true,
    kSecReturnAttributes: true
  ] as CFDictionary
  
  var ref: AnyObject?
  
  let status = SecItemAdd(keychainItem, &ref)
  let result = ref as! NSDictionary
  print("Operation finished with status: \(status)")
  print("Username: \(result[kSecAttrAccount] ?? "")")
  let passwordData = result[kSecValueData] as! Data
  let passwordString = String(data: passwordData, encoding: .utf8)
  print("Password: \(passwordString ?? "")")
}
```

To query the keychain and retrieve the items, we use the `SecItemCopyMatching` function. The two parameters it takes are actually the same ones as `SecItemAdd`. The way these two functions is the same: You specify a query, and get something in return, including nil. it returns a `OSStatus` like `SecItemAdd`.

While `SecItemCopyMatching` has an optional second parameter, you probably want to specify it most of the time, otherwise you will never get any data back when working with it.

You write your query the same way you did when creating items, but there's a few things we need to keep in mind. First, the queries can be as specific or as generic as you want. Second, you can force the keychain to return one or more items at once.

Once again this API is messy, because the `result` can once again return more than one type. The conditions are the same as when returning data from `SecItemAdd`, but there is one more type `SecItemCopyMatching` can return. If you specify the key `kSecMatchLimit` and give it a value bigger than `1`, you will get a `CFArray` of `CFDictionary`.

Let's see this in action. You know you have many items that have `pullipstyle.com` as their `kSecAttrServer`. Despite that, the following query will only return one:

```swift
let query = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrServer: "pullipstyle.com",
  kSecReturnAttributes: true,
  kSecReturnData: true
] as CFDictionary

var result: AnyObject?
let status = SecItemCopyMatching(query, &result)

print("Operation finished with status: \(status)")
let dic = result as! NSDictionary

let username = dic[kSecAttrAccount] ?? ""
let passwordData = dic[kSecValueData] as! Data
let password = String(data: passwordData, encoding: .utf8)!
print("Username: \(username)")
print("Password: \(password)")
```

If the query doesn't find any items to return, `result` will be nil, so make sure you check for that.

To return more than one, we will specify `kSecMatchLimit` to a high enough number to return all the entries:

```swift
let query = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrServer: "pullipstyle.com",
  kSecReturnAttributes: true,
  kSecReturnData: true,
  kSecMatchLimit: 5
] as CFDictionary

var result: AnyObject?
let status = SecItemCopyMatching(query, &result)

print("Operation finished with status: \(status)")
let array = result as! [NSDictionary]

array.forEach { dic in
  let username = dic[kSecAttrAccount] ?? ""
  let passwordData = dic[kSecValueData] as! Data
  let password = String(data: passwordData, encoding: .utf8)!
  print("Username: \(username)")
  print("Password: \(password)")
}
```

Now the code has been adapted to work with an array instead of a dictionary, and we can now get all the entries that match the query.

One annoying detail of specifying `kSecMatchLimit` is that when your query produces zero results, `result` will again be `nil` instead of an empty array, so keep that in mind.

The more attribute keys you add, the more specific your queries become.

```swift
let query = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrServer: "pullipstyle.com",
  kSecAttrAccount: "andyibanez",
  kSecReturnAttributes: true,
  kSecReturnData: true,
  kSecMatchLimit: 5
] as CFDictionary

var result: AnyObject?
let status = SecItemCopyMatching(query, &result)

print("Operation finished with status: \(status)")
let array = result as! [NSDictionary]

array.forEach { dic in
  let username = dic[kSecAttrAccount] ?? ""
  let passwordData = dic[kSecValueData] as! Data
  let password = String(data: passwordData, encoding: .utf8)!
  print("Username: \(username)")
  print("Password: \(password)")
}
```

The above code will match both the server (`pullipstyle.com`) and the username (`andyibanez`). This returns one item only in our specific case. Luckily since `kSecMatchLimit` is bigger than one, we still get an array and not a dictionary.

```
Operation finished with status: 0
Username: andyibanez
Password: andyibanez-Pullip2020
```

## Updating Items

To update items in the keychain, you use the `SecItemUpdate` function which also returns an `OSStatus`. Unlike `SecItemAdd` and `SecItemCopyMatching`, this function takes two `CFDictionaries` as its arguments. The first one is the query which you know and love. The second one contains the attributes you want to update and their new values.

When specifying the query, you really need to be careful with how specific it is. If it isn't specific enough, you may end up updating multiple items at once when you didn't intend to.

```swift
let query = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrServer: "pullipstyle.com",
] as CFDictionary

let updateFields = [
  kSecValueData: "newPassword".data(using: .utf8)!
] as CFDictionary

let status = SecItemUpdate(query, updateFields)
print("Operation finished with status: \(status)")
```

This will successfully modify the password of all `kSecClass`es whose `kSecAttrServer` is `pullipstyle.com`. It will update ALL the entries, because our query is too general and it matches many entries. Always try to make the query more specific to avoid these problems, especially if your keychain stores a lot of sensitive data.

In this case it would better to also specify the username (`kSecAttrAccount`) of the user we want to update:

```swift
let query = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrServer: "pullipstylew.com",
  kSecAttrAccount: "andyibanez"
] as CFDictionary

let updateFields = [
  kSecValueData: "newPassword".data(using: .utf8)!
] as CFDictionary

let status = SecItemUpdate(query, updateFields)
print("Operation finished with status: \(status)")
```

## Deleting Items

To delete an item, use the `SecItemDelete` function. This function only takes one parameter, the query, and returns an `OSStatus`.

Just like when we are updating items, make sure your query is very specific so you only delete what you intend to delete. If you don't you may delete extra entries accidentally.

```swift
let query = [
  kSecClass: kSecClassInternetPassword,
  kSecAttrServer: "pullipstyle.com",
  kSecAttrAccount: "andyibanez.com"
] as CFDictionary

SecItemDelete(query)
```

# Conclusion

Using the keychain to store sensitive data is the way to go. While it is a low-level API, the bridging to Swift has become more bearable in the last few years. You generally won't worry too much about the bridging types, but it's worth keeping into amount that the operations that return data back can return different data types.

We explored the basic usage of the keychain services APIs. We learend how to add, search, update, and delete entries, but the keychain can actually do quite a lot more than just that. Hopefully this article helped you get started to write more secure storage for sensitive data in your own apps.

