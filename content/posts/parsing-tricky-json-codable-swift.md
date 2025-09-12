---
title: "Parsing Tricky JSON With Codable in Swift"
date: 2020-10-28T07:00:00-04:00
originalDate: 2020-10-26T10:08:15-04:00
publishDate: 2020-10-28T07:00:00-04:00
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
 - codable
categories:
 - development
description: "Learn how to deal with tricky situations when parsing JSON with Swift's Codable."
keywords:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - macos
 - tvos
 - watchos
 - codable
---

If you have been writing Swift in the past couple of years, you have probably been using [`Codable`](https://developer.apple.com/documentation/swift/codable) (which is really just the composition of [`Decodable`](https://developer.apple.com/documentation/swift/decodable) and [`Encodable`](https://developer.apple.com/documentation/swift/encodable) in the same protocol).

If you have been writing iOS apps for longer, you likely know about [`JSONSerialization`](https://developer.apple.com/documentation/foundation/jsonserialization) as well, which is the backbone of `Codable` and it allows you to do more manual work when parsing JSON, seemingly giving you more control.

If you know `JSONSerialization`, you have probably found times in which Codable seemingly doesn't give you the flexibility you need, and you may have been tempted to drop `Codable` in favor of `JSONSerialization` when parsing very specific or even corrupted JSON.

`Codable` is actually more powerful than you expect, and if you know how to use it fully, you will never need to drop down to `JSONSerialization` for those cases when `Codable` seems like it's holding you back.

In this article, we will explore one feature of `Codable` that makes it parsing tricky JSON possible by exploring two specific scenarios:

* When you have a field that seems to return different data types in different situations, and
* when you have a field that is a collection such as an array or a dictionary, but the datatype within this collection varies.

My intention is to show you these specific situations because I have dealt with them in the real world, and because the methods you will learn here to can give you other ideas for working with different cases of "malformed" (but valid) JSON.

# Tricky JSON

Unless you can have any kind of influence over the backend, you shouldn't really expect JSON to be perfect (and actually, my own experience at my job has shown me that there's times even when you can ask for backend changes, it's not possible to be done, or not worth it). There's many cases in which Codable can already help you. being able to declare properties as optionals already does a huge job dealing with missing properties. So even if a JSON returns a field that should exist, there are times when you can get the job done providing a default value yourself.

But when our JSON has unexpected different datatypes, things can be messy. Even if you have a field marked as an optional, if `Codable` finds it, it will try to parse it with that datatype and throw an error if it is something else.

## Dealing With Different Datatypes

When I started my current job, I started migrating some very old legacy Objective-C code into Swift. One of these tasks involved migrating an Objective-C parser that relied on `JSONSerialiation` to parse content into our objects. We never needed to operate on this field. Essentially we had to receive it, and pass it back to the backend as-is.

Because of this, and the nature of `JSONSerialization`, nobody ever realized that this value was sometimes returning as a string, and sometimes as a number. It really didn't matter. Below is an example of the object in question and the kind of data it returned. This is not the real code I found at my job, but it's very easy to recreate:

```text
{
	"username": "aibanez",
	"phone_number": 1234567,
	"identifier_hash": "ABXAASDASFASFS"
}
```

I called this object `UserInfo`, and it returned as a nested object in multiple calls, such as:

```text
/last_login_info

{
	"login_date": "2020-05-05T05:00:00-04:00",
	"country": "Bolivia",
	"ip_address": "192.168.0.1",
	"user": {
		"username": "aibanez",
		"phone_number": 1234567,
		"identifier_hash": "ABXAASDASFASFS"
	}
}
```

```text
/login

{
	"process_token": "ABCASD",
	"previous_device_name": "iPhone 11 Pro Max",
	"user": {
		"username": "aibanez",
		"phone_number": "1234567",
		"identifier_hash": "ABXAASDASFASFS"
	}
}
```

The first call is used to retrieve the last session information when you launch the app. If you download the app on a new device, the app calls the last method and returns that JSON.

You can clearly see the red flag here. `phone_number` is an integer in one call, and a string in another!

In the beginning this was problem because I told the backend about this inconsistency and they wouldn't fix it. Also note that this object in real life is much more expansive. Tens of fields in one request. The quick solution at the time was to create two classes for `UserInfo` - `UserInfoString`, and `UserInfoInt`. Luckily I had a bit of time to research a real solution.

I started by declaring `UserInfo` as such:

```swift
class UserInfo: Codable {
    let username: String
    let phoneNumber: Int
    let identifier: String
    
    enum CodingKeys: String, CodingKey {
        case username = "username"
        case phoneNumber = "phone_number"
        case identifier = "identifier_hash"
    }
}
```

And the two objects that had this nested object:

```swift
class LastLogin: Codable {
    let loginDate: String
    let country: String
    let ipAddress: String
    let user: UserInfo
    
    enum CodingKeys: String, CodingKey {
        case loginDate = "login_date"
        case country = "country"
        case ipAddress = "ip_address"
        case user = "user"
    }
}

class LoginInfo: Codable {
    let processToken: String
    let previousDeviceName: String
    let user: UserInfo
    
    enum CodingKeys: String, CodingKey {
        case processToken = "process_token"
        case previousDeviceName = "previous_device_name"
        case user = "user"
    }
}
```

At this point, we can only parse one of them. Whoever has the phone number as a string will fail (the JSON returned by `/login`).

To solve this, you can manually implement the required initializer provided by `Decodable`. Inside it you can parse each expected field, one by one.

This is how the initializer was implemented in `UserInfo`:

```swift
required init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    self.username = try container.decode(String.self, forKey: .username)
    self.identifier = try container.decode(String.self, forKey: .identifier)
    // Try to parse the phone number as an int first.
    do {
        self.phoneNumber = try container.decode(Int.self, forKey: .phoneNumber)
    } catch {
        // Parsing it as an int failed. We will try to parse it as a string.
        let phoneString = try container.decode(String.self, forKey: .phoneNumber)
        if let phoneInt = Int(phoneString) {
            self.phoneNumber = phoneInt
        } else {
            throw error
        }
    }
}
```

In my particular case, I knew `username` and `identifier_hash` were always going to return strings, which is why I just parse those two fields directly.

When attempting to parse the phone number, the parser first tries to parse it as an `Int` as that is the datatype in the model. If that fails, we will try to parse it into a temporary String variable. We then try to convert it into into an integer - if it succeeds, we will assign the variable and go on with on with our day. If it fails, we will rethrow the error telling us `phone_number` expected a String, but found something else instead. If the error is rethrown, we will have to look into the JSON and see what it is returning. This is not likely to happen in this specific case, but it *could* if you were parsing a field that expected a number but suddenly returned floating points or even strings.

Also, keep in mind that if your object had optional fields, you can use `container.decodeIfPresent` instead of `container.decode`. This will allow nil values to be ignored, though errors will be thrown if the value does exist and it's of an unexpected data type.

## Dealing with Different Datatypes Within Collections

I found this "tricky JSON" situation working on my weekend app. My app, Silvianna, is a client for a website called [Anilist](https://www.anilist.co) - an anime and manga database where you can search, find, and discover new anime to watch or manga to read.

They use a GraphQL API, but due to implementation details on their side, I couldn't just parse the responses using something like [Apollo](https://www.apollographql.com). Instead, I created objects for everything I wanted to parse.

One specific response returned a dictionary like this:

```swift
{
	"advancedScores": {
		"Story": 0,
		"Characters": 0,
		"Visual": 0,
		"Audio": 0,
		"Enjoyment": 0
	}
}
```

Users can configure their own advanced scoring parameters, so I had to parse this as a dictionary of type `[String: Double]`.

Anilist is supposed to return Doubles here, but I discovered when parsing a huge array that contained this nested object, that there was a case in which it returned something like this instead:

```swift
{
	"advancedScores": {
		"Story": "0",
		"Characters": 0,
		"Visual": 0,
		"Audio": 0,
		"Enjoyment": 0
	}
}
```

For reasons entirely unknown to me (and to the people who had worked with the Anilist API), the "Story" key was returning with its value as a String. This only happened in one object in a gigantic array of around 900 objects that had this nested object.

To deal with this, I made the assumption that the values here are always floating points. My decision was backed up by the Anilist docs and by the community who had used the API.

Before I stumbled upon this problem, my model looked like this (simplified):

```swift
class UserMediaEntry: Codable {
    let advancedScores : [String: Double]
}
```

In order to parse the value of the dictionary, I ended up creating an intermediary object called `WrappedDouble`.

The `Decoder` object we receive from the `required init(from decoder: Decoder)` has one more useful container: `singleValueContainer()`. We can use it to decode a single value without having to pass in the `CodingKey`s or anything like that.

The implementation of wrapped value is as follows:

```swift
class WrappedDouble: Codable {
    let value: Double
    
    required init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        do {
            // Try to parse it as a Double
            value = try container.decode(Double.self)
        } catch {
            let tempString = try container.decode(String.self)
            if let convertedDouble = Double(tempString) {
                value = convertedDouble
            } else {
                throw error
            }
        }
    }
}
``` 

This parser will take in a single value, and the rest of the logic is pretty straightforward: We try parsing this object as a `Double` first, and if that fails, we try to parse it as a String.

The `UserMediaEntry` object above now needs to be modified. We will also do slightly more manual parsing. We are not going to modify the datatype of `advancedScores` at all. Instead we will parse the `Double`s out using our `DoubleWrapper` object, get the value, and finish creating `UserMediaEntry` with them:

```swift
class UserMediaEntry: Codable {
    required init(from decoder: Decoder) throws {
        let decoder = try decoder.container(keyedBy: CodingKeys.self)
        let wrappedDoubleDic = try decoder.decode([String: WrappedDouble].self, forKey: .advancedScores)
        advancedScores = wrappedDoubleDic.mapValues { $0.value }
    }
    
    let advancedScores : [String: Double]
}
```

Pretty straightforward. Once we have our `DoubleWrapper` object, we can try to parse a given key using `[String: DoubleWrapper]`. `DoubleWrapper` can get a double out of a `Double` itself or a String. If our init can parse that dictionary, we than map it to a new dictionary keeping the keys, but transforming them to Double instead.

(*Aside note*: `Dictionary.mapValues` will map all the dictionary values keeping their keys, so it's perfect to convert our `DoubleWrapper` into `Double` without any issues).

# Conclusion

JSON is oftentimes a format that is out of our control. Luckily `Codable` actually provides all the tools we need to parse extravagant JSON responses without having to drop down to `JSONSerialization`. Often times when dealing with broken (but valid) JSON, the first solution we may think of is to use the lower level APIs, but by manually overriding `init(from)`, we can do manual parsing even easier.

