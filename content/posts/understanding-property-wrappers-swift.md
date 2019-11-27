---
title: "Understanding Property Wrappers Swift"
date: 2019-11-23T16:26:13-04:00
draft: false
publishDate: 2019-11-27T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - property wrappers
categories:
 - development
description: "Learn to build URLs Swiftly with NSURLComponents."
keywords:
 - swift
 - programming
 - apple
 - property wrappers
---

Swift 5.1 introduced a sleuth of wonderful features, and amongst them, there's one that is essential for SwiftUI: Property Wrappers. Property wrappers are a powerful feature in Swift that allow you to wrap behavior along with properties. This allows us to do some interesting things. If you have seen SwiftUI, you've seen the `@State` "keyword", and you know that it allows you to modify structs. This is possible thanks to the behavior "wrapped" within properties marked with it.

But `@State` isn't really a keyword. It's just functionality exposed to us through Property Wrappers, and just like SwiftUI provides `@State`, `@EnvironmentObject`, and more, you can create your own property wrappers for your own use, and they can let you, amongst other things, simplify the code you write and get rid of a lot of boilerplate code. Also, the use of property wrappers is not limited to SwiftUI, and you can find uses for them in any context.

In fewer words, we can define property wrappers as a layer between how a property is stored and the code that defines a property. <sup>[1](https://docs.swift.org/swift-book/LanguageGuide/Properties.html)</sup>

Property wrappers are useful in any context and you shouldn't constraint yourself in using them only with SwiftUI. In this article we will explore some ideas that can be implemented with property wrappers that can be done without using SwiftUI at all.

# A Gotcha

Before we get started, it's worth mentioning that you can only apply *one* property wrapper to a given property at any given time. You cannot, for example, do something like `@State @EnvironmentObject var foo = Foo()`.

# Playing with Property Wrappers

## Implementing Your Own Property Wrappers

To implement your own property wrapper, you need to do a few things:

* Declare a struct that will wrap the property.
* Mark the struct as a `@propertyWrapper` before the `struct` keyword.
* Implement the `var wrappedValue` computed variable. You can use `get` to get the value of the property, and `set` to set it. With this, you can see that you can let a property wrapper store its value anywhere.

To exemplify this, we will write a simple property wrapper that works with `String`s and it changes them to uppercase letters.

```swift
@propertyWrapper
struct Capitalized {
  private(set) var text: String = ""
  var wrappedValue: String {
    get { return text }
    set { text = newValue.uppercased() }
  }
}
```

This is a very simple wrapper with a straight forward task, but the complexity can grow depending on what you want to do.

Our struct `Capitalized` *is* our new property wrapper. It will store the string internally in the `text` property, but this is not a requirement (you could store it in a database, cache, or anywhere else). The `wrappedValue` property will handle the storage and retrieval for the contents of the property. In this case, when we return the property, we will return the standard property, and when we store it, we will make it an uppercase string. Nothing prevents you from doing it the other way around - storing the string as-is, and returning it as `uppercased()` -, so feel free to explore and to implement it differently as necessary depending on the context.

Notice that our `text` variable needs a default value in this case. If you don't set it, you will have problems compiling the code above.

## Using Custom Property Wrappers

Now that we defined our property wrapper, we can finally apply it to members of a struct or class. To show how to do this, we will create a struct called `Name` that will store the first and last name of someone, and they will be stored in capital letters only:

```swift
struct Name {
  @Capitalized var firstName: String
  @Capitalized var lastName: String
}
```

And now, when you create an object of this type and set its properties, they will be uppercased when you need them:

```swift
var myName = Name()
myName.firstName = "andy"
myName.lastName = "ibanez"
print(myName.firstName) // prints "ANDY"
print(myName.lastName) // prints "IBANEZ"
```

## Advanced Usage

What we saw above was a very simple wrapper that changes the capitalization of strings. But we can do a few more interesting things with them that open the door to more interesting ideas.

You can, for example, pass in parameters to the wrapper property itself. This allows you to configure how the property should behave on the fly.

As an example, we will write a new property wrapped, `ConfigurableCapitalization`, that allows us to specify how a `String` should be capitalized. We can specify if we want it to be `uppercased`, `lowercased`, or `capitalized`.

```swift
@propertyWrapper
struct ConfigurableCapitalization {
  
  enum Settings {
    case uppercased
    case capitalized
    case lowercased
  }
  
  private(set) var text: String = ""
  public let setting: Settings
  
  var wrappedValue: String {
    get { return text }
    set {
      switch setting {
      case .capitalized: text = newValue.capitalized
      case .lowercased: text = newValue.lowercased()
      case .uppercased: text = newValue.uppercased()
      }
    }
  }
}

struct FullName {
  @ConfigurableCapitalization(setting: .capitalized) var firstName: String
  @ConfigurableCapitalization(setting: .uppercased) var lastName: String
}

var myFullName = FullName()
myFullName.firstName = "andy"
myFullName.lastName = "ibanez"
print(myFullName.firstName) // prints "Andy"
print(myFullName.lastName) // prints "IBANEZ"
```

First, we create our `ConfigurableCapitalization` property wrapped, which contains an `enum` that lets us specify the capitalization of a String. In the setter of the `wrappedValue`, we instruct the code to store the property in any of the specified capitalization types.

Then we created a `FullName` object that will store a `firstName` as a `capitalized` string (it will capitalize the first letter of each word), and the last name as an `uppercased` string.

Finally, we assign some values to these property and print their values to the see the results.

# Projecting Values from Property Wrappers

Property wrappers can expose even more functionality through the use of Projected Values. The projected value is implemented as a property in the property wrapper called `projectedValue`, and it can be of any type you want. You can use this for many things. In our example, we will simply use it to tell us what `Setting` it used to capitalize the string.

```swift
@propertyWrapper
struct ConfigurableCapitalization {
  
  var projectedValue: Settings = .capitalized
  
  enum Settings {
    case uppercased
    case capitalized
    case lowercased
  }
  
  private(set) var text: String = ""
  public let setting: Settings
  
  var wrappedValue: String {
    get { return text }
    set {
      switch setting {
      case .capitalized: text = newValue.capitalized
      case .lowercased: text = newValue.lowercased()
      case .uppercased: text = newValue.uppercased()
      }
      projectedValue = setting
    }
  }
}
```

```swift
print(myFullName.$firstName) // prints "capitalized"
```

But you can use it for me. If you are storing your values in a database, you can expose the connector object to do something with it, like flush values, or more.

# The Downsides

Property wrappers are an amazing feature, but they have one downside: They provide too much "black magic" for someone who is not familiar with them. If you handed a new programmer our `FullName` class and instructed them to use it, they may be surprised at what's going on behind the scenes. This can only be solved with proper documentation.

# Conclusion

Property wrappers are a very interesting Swift feature. While their usage is prominent in SwiftUI, it doesn't have to be. It can be hard to wrap (heh) your head around them at first, but once you take the time to understand them, you can see they offer ways to simplify, reduce boilerplate, or simply wrap a lot of power in a small container.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.