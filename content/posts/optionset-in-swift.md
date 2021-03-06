---
title: "OptionSet in Swift"
date: 2021-02-17T07:30:00-04:00
originalDate: 2021-02-13T22:30:52-04:00
publishDate: 2021-02-17T07:30:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - apple
 - swift
 - programming
 - optionset
keywords:
 - apple
 - swift
 - programming
 - optionset
categories:
 - development
description: "Learn what OptionSet is in Swift and how to use it."
---

Creating configurable APIs for other developers can be a fun task. But depending on what languages and tools you are using, you may sometimes create customizable APIs that are more pleasant than others.

In today's article, we will explore a tiny feature in Swift that allows us to create configurable APIs easily that are a joy to use by other developers: OptionSet.

# Introducing OptionSet

Like its name implies, an OptionSet gives us a group of options. These options are pre-defined for our users, and when create an OptionSet, our users are constrained to using the values we are providing within.

## OptionSet Throughout Apple's APIs

We will create our own OptionSets soon, but before we do, let's explore a few areas you may have seen them being used across Apple's frameworks. It's very likely you have used them without noticing it.

### NSJSONSerialization

If you have been parsing JSON before Codable was a thing, it's possible you have used the `NSJSONSerialization` object. For example, to create an object from `Data`:

```swift
let json = JSONSerialization.jsonObject(with: data, options: .allowFragments)
```

Focusing on the `options` parameter (of type `JSONSerialization.ReadingOptions`), it is simply a `struct` with the following values:

```swift
        public static var mutableContainers: JSONSerialization.ReadingOptions { get }

        public static var mutableLeaves: JSONSerialization.ReadingOptions { get }

        public static var fragmentsAllowed: JSONSerialization.ReadingOptions { get }

        @available(iOS, introduced: 5.0, deprecated: 100000, renamed: "JSONSerialization.ReadingOptions.fragmentsAllowed")
        public static var allowFragments: JSONSerialization.ReadingOptions { get }
```

The thing about OptionSet is that it can take either none, one, or many options. You may think, looking at the syntax, that it can only use one at a time (in the example above, we are using `.allowFragments`), but an option set can actually take multiple values if you pass in an array-like value.

```swift
let json = JSONSerialization.jsonObject(with: data, options: [.allowFragments, .mutableContainers, .mutableContainers])
```

Finally, if you don't want to pass in any options, you need to specify an empty array.

```swift
let json = JSONSerialization.jsonObject(with: data, options: [])
```

It's hard to know at first glance if you are using an option set or just passing in some random enumerators, so the documentation across Apple's frameworks will come in handy.

## Creating your own OptionSet

Of course, OptionSets wouldn't be half as interesting unless you were able to create your own.

To create your own option set, you need to conform to the `OptionSet` protocol and specify. The protocol will constrain you to requiring a `rawValue` property.

```swift
struct Currency: OptionSet {
  let rawValue: Int
  
  static let bolivianBoliviano = Currency(rawValue: 1 << 0)
  static let argentinianPeso = Currency(rawValue: 1 << 1)
  static let chileanPeso = Currency(rawValue: 1 << 2)
  static let usd = Currency(rawValue: 1 << 3)
  static let canadianDollar = Currency(rawValue: 1 << 4)
  static let mexicanPeso = Currency(rawValue: 1 << 5)
}
```

For the sake of efficiency, the raw value is an `Int`. These Ints are increasing in powers of 2, and library will internally optimize storage for the option set. If you try to use a different datatype for the raw value, you will likely encounter an error saying that your OptionSet must conform to `SetAlgebra`. We will explore `SetAlgebra` a little bit later.

You can even go ahead and create option combinations for popular options. In the example above, we can create groupings for South American currencies and North American currencies.

```swift
  static let southAmericanCurrencies: [Currency] = [.bolivianBoliviano, .argentinianPeso, .chileanPeso]
  static let northAmericanCurrencies: [Currency] = [.usd, .canadianDollar, .mexicanPeso]
  
  static let all: [Currency] = Self.southAmericanCurrencies + Self.northAmericanCurrencies
```

The final declaration looks like this:

```Swift
struct Currency: OptionSet {
  let rawValue: Int
  
  static let bolivianBoliviano = Currency(rawValue: 1 << 0)
  static let argentinianPeso = Currency(rawValue: 1 << 1)
  static let chileanPeso = Currency(rawValue: 1 << 2)
  static let usd = Currency(rawValue: 1 << 3)
  static let canadianDollar = Currency(rawValue: 1 << 4)
  static let mexicanPeso = Currency(rawValue: 1 << 5)
  
  static let southAmericanCurrencies: [Currency] = [.bolivianBoliviano, .argentinianPeso, .chileanPeso]
  static let northAmericanCurrencies: [Currency] = [.usd, .canadianDollar, .mexicanPeso]
  
  static let all: [Currency] = Self.southAmericanCurrencies + Self.northAmericanCurrencies
}
```

### The Necessity of SetAlgebra

If you tried making your OptionSet have a different raw value that isn't an `Int`, you have likely found yourself being yelled at by the compiler for not conforming to SetAlgebra.

If you don't understand the `Set` data structure found in Swift and virtually all common libraries of other languages, I recommend you read [my article on sets](https://www.andyibanez.com/posts/understanding-basic-data-structures-swift-sets/). The advantages of having an option set conform to `SetAlgebra` is that we can do set operations in order to grab the OptionSet data when we need to operate with it.

You can, for example, do the simplest operation, which is checking if a set contains a given option:

```Swift
Currency.southAmericanCurrencies.contains(.bolivianBoliviano)
```

To more complex set operations, such as joins, checking set membership in bigger sets, and more.

## Using your Own OptionSets

Finally, to use your OptionSet you just treat it as a set. When a method takes a `Currency`, it's taking a set. You can use that set to perform operations based on the values provided for said set.

```swift
func convertUSDTo(_ currencies: Currency, value: Double) -> [CurrencyPair] {
  // Grab the values of the set and operate on it
  if currencies.contains(.bolivianBoliviano) {
    // calculate bob
  }
  
  if currencies.contains(.chileanPeso) {
    // Calculate
  }
  //...
}
```

# Conclusion

Building configurable APIs is not necessarily a challenge, but it can always be improved. OptionSet provides us with a way to let user configure their calls to our APIs and internally they are essentially sets.

