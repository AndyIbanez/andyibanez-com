---
title: "Nsuserdefaults Property Wrappers"
date: 2019-12-04T07:00:00-04:00
originalDate: 2019-12-01T17:25:36-04:00
publishDate: 2019-12-04T07:00:00-04:00
draft: true
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - property wrappers
 - programming
 - apple
 - ios
 - macos
 - tvos
 - watchos
 - ipados
categories:
 - development
description: "Learn to use and understand the Result type in Swift."
keywords:
 - swift
 - property wrappers
 - ios
 - tvos
 - ipados
 - watchos
---

Last week we talked about Property Wrappers, what they are, and how they work with Swift. In this article, we will build upon that to write a very nice property wrapper for user settings based on `NSUserDefaults`.

# But Why?

I don't know about you, but a pattern I see *a lot* (and I'm guilty of doing this myself) is to just wrap all user defaults in a Singleton. This works, but singletons in general are not a pattern everyone is particularly a fan of. Singletons can grow, and it can be a pain to maintain if they have too many properties. How do you logically separate the properties? Does it even make sense to wrap a singleton around everything related to user defaults?

For this reason, people have devised different ways to deal with user defaults, and I'm going to show you a new one.

## The Advantages

I found more pros than cons when it comes to using wrapped properties for use defaults, including:

* It's more obvious to see what settings are relevant in different view controllers. In this architecture, we will write a property wrapper for User Defaults, and we will use it for all defaults that come to mind. This way, we can create properties that represent said setting, and then you will know what settings are relevant in different view controllers or other areas of your app. For example, suppose you have an app where you can configure a default locale, calendar type, timezone, if the app should be locked with FaceID when entering the background, and a currency type. Then you have a `CalendarViewController` where the user can see his configured locale, timezone, and calendar type. Whether the app should lock and currency types are irrelevant here. By treating them as properties, you can put them at the top of the class and the next developer who maintains the code will know what defaults are relevant for that screen:

```swift
class CalendarViewController: UIViewController {
  @UserDefault(key: .calendarType) var calendarType: String
  @UserDefault(key: .timezone) var timezone: String
  @UserDefault(key: .locale) var locale: String
  
  //...
}
```

* You don't have to maintain very large singleton files for your settings. Instead you just have to write a property wrapper file and never concern yourself with it again.

## The Disadvantages

There is one disadvantage that I was able to find with this method, so if you find a good way to deal with it, let me know, I'm more than happy to hear potential ideas for this.

There is no easy way to write testable code with this. You can pass in a `UserDefaults` object to each property, but this is may not be the best idea if you are these wrapped properties in many places.

# Property Wrappers for User Defaults

At the end of this tutorial, we will have two different property wrappers for settings, but you will essentially write them once and do small modifications to them when necessary.

## The `UserDefault` Property Wrapper

This property wrapper will be used to deal with standard data types supported by `UserDefaults`. In other words, it will be compatible with `String`s, `Int`s, `Bool`s, `Data`, and others that work with UserDefaults by default.

Start by writing this skeleton:

```swift
@propertyWrapper
struct UserDefault<T> {
}
```

We want it to be generic because user defaults can store many different data types. By making it generic, it can support any data type that user defaults supports.

Then, we are going to add a few properties, and an enum:

```swift
@propertyWrapper
struct UserDefault<T> {
  enum Key: String {
    case lockOnExit = "lock_on_exit"
    case showImages = "show_images"
  }

  let userDefaults: UserDefaults
  let key: Key
  let defaultValue: T
}
```

The `Key` enum will take keys that will be used to retrieve the data from `UserDefaults` internally. You can choose to not use this and just pass in the string keys, but I prefer to have an enum because I get autocomplete and it's harder to make mistakes when dealing with defaults.

As for the properties, we will inject a `UserDefaults` object and we will provide one by default when the user does not specify one. The `key` property holds the key of the default we want to retrieve. Finally, we define a default value to use when the key we provided does not exist in the underlying user defaults.

 Next, implement a simple initializer for the property wrapper. We will force the user to provide a key, but the default value and underlying `UserDefaults` object are optional:
 
```swift
  init(userDefaults: UserDefaults = UserDefaults.standard,
       key: Key,
       defaultValue: T) {
    self.userDefaults = userDefaults
    self.key = key
    self.defaultValue = defaultValue
  }
```

Finally, implement the `wrappedValue` calculated property. This will do the magic of retrieving and saving data to `UserDefaults`:

```swift
  var wrappedValue: T {
    get { return userDefaults.object(forKey: key.rawValue) as? T ?? defaultValue }
    set { userDefaults.set(newValue, forKey: key.rawValue) }
  }
```

We now have a fully functional property wrapper for standard UserDefault values. For reference, the full implementation is below:

```swift
@propertyWrapper
struct UserDefault<T> {
  enum Key: String {
    case lockOnExit = "lockOnExit"
    case showImages = "show_images"
  }
  
  let userDefaults: UserDefaults
  let key: Key
  let defaultValue: T
  
  init(userDefaults: UserDefaults = UserDefaults.standard,
       key: Key,
       defaultValue: T) {
    self.userDefaults = userDefaults
    self.key = key
    self.defaultValue = defaultValue
  }
  
  var wrappedValue: T {
    get { return userDefaults.object(forKey: key.rawValue) as? T ?? defaultValue }
    set { userDefaults.set(newValue, forKey: key.rawValue) }
  }
}
```

Using it is very easy:

```swift
class ImagesViewController: UIViewController {
  @UserDefault(key: .lockOnExit, defaultValue: true) var maxAttempts
  @UserDefault(key: .showImages, defaultValue: false) var showImages
}
```

Now that you have this class, your user defaults are more obvious and it's easier to know what context they should be used in.

## The `ComplexUserDefault` Property Wrapper

It's common to store more complex data in UserDefaults, such as complete JSON structures, or just complete objects. To handle these cases, I created another property wrapper called `ComplexUserDefault` which serializes objects into `Data` using `Codable` and persists them that way. There are many ways you could do this, but I found this one was better and more self contained than the alternatives (like using protocols and extensions).

This property wrapper looks very similar to the previous one, but with a few changes. First, you cannot specify a default value because I found it doesn't make sense in this case. So this property wrapper can return and store nil values. Then, the `wrappedProperty` can return nil and it takes care of the serialization and deserialization of values. Finally, the generic value is constrained to objects that conform to `Codable`.

The complete implementation looks like this:

```swift
@propertyWrapper
struct ComplexUserDefault<T: Codable> {
  enum Key: String {
    case userInfo = "user_info"
  }
  
  let userDefaults: UserDefaults
  let key: Key
  
  init(userDefaults: UserDefaults = UserDefaults.standard,
       key: Key) {
    self.userDefaults = userDefaults
    self.key = key
  }
  
  var wrappedValue: T? {
    get {
      guard let data = userDefaults.data(forKey: key.rawValue) else { return nil }
      let object = try? JSONDecoder().decode(T.self, from: data)
      return object
    }
    
    set {
      guard let object = newValue else { return }
      let data = try? JSONEncoder().encode(object)
      userDefaults.set(data, forKey: key.rawValue)
    }
  }
}
```

And to show how it works, let's create a `UserInfo` object which will be stored in the `user_info` key:

```swift
struct UserInfo: Codable {
  let username: String
  let email: String
  let firstName: String
  let lastName: String
}
```

Using it as a property is the same as any other property wrapper:

```swift
class UserProfile {
  @ComplexUserDefault(key: .userInfo) var userInfo: UserInfo?
}
```

And finally, assigning the property is nothing different:

```swift
let profile = UserProfile()
profile.userInfo = UserInfo(username: "aibanez",
                            email: "andy@andyibanez.com",
                            firstName: "Andy",
                            lastName: "Ibanez")
```

You now have two property wrappers to deal with your settings in a clean and independent way. You don't have to fight with singletons for your defaults ever again.

# Conclusion

Property Wrappers are very powerful, and they can help you kill common patterns in favor of something nicer and more contextually aware. Using them for user defaults has a lot of benefits and it helps you write cleaner code, not to mention it can help new programmers in a project get up to speed with how defaults are stored.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.