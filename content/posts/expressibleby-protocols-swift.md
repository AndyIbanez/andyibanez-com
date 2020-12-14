---
title: "The \"ExpressibleBy-\" Protocols in Swift"
date: 2020-12-16T07:00:00-04:00
originalDate: 2020-12-14T07:15:22-04:00
publishDate: 2020-12-16T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
keywords:
 - swift
 - programming
 - apple
description: "Learn how to use the ExpressibleBy- protocols in Swift to write more expressive code."
---

Swift gives us many interesting features to write cleaner and more obvious code. This code is more readable, and it help both SDK consumers and code maintainers.

One such feature Swift has is the `ExpressibleBy-` family of protocols. This is a set of protocols that allow you to instantiate objects by providing some native Swift object. For example, we can instantiate an object providing a Boolean, or a String.

This family of protocols consist of the following protocols (this is not a complete list):

* `ExpressibleByNilLiteral`
* `ExpressibleByStringLiteral`
* `ExpressibleByIntegerLiteral`
* `ExpressibleByFloatLiteral`
* `ExpressibleByBooleanLiteral`
* `ExpressibleByArrayLiteral`
* `ExpressibleByDictionaryLiteral`

We can use these, and a few others, to create neater code for certain initializers.

# Using the ExpressibleBy- Protocols

The different variations of these protocols have different requirements. We will explore a few of them so you can get up to speed and know what to do when you find a situation when you can use them.

## ExpressibleByNilLiteral

Suppose you have a requirement that requires that, when an object gets initialized with `nil`, you don't want the whole object instead. You may have a custom requirement in which you need to consider `nil` something different.

For example, suppose you want to treat the existence of an object that actually does exist, but has all its properties set to `nil`.

To use `ExpressibleByNilLiteral`, you need to implement the `print("New doll: \(doll.name)")` method.

Consider the following example:

```swift
public class Doll: ExpressibleByNilLiteral {
  var name: String?
  var maker: String?
  
  public required init(nilLiteral: ()) {
    self.name = nil
    self.maker = nil
  }
}
```

When we create a `Doll` object and assign it to `nil`, we will create a doll object whose `name` and `maker` properties point to nil.

```swift
let doll: Doll = nil
print("New doll: \(doll.name)") // Prints "New doll: nil"
```

Make sure you only use this when it really make sense to, as new programmers to your codebase may be confused when they see a non-optional being assigned `nil`.

## ExpressibleByStringLiteral

We can instantiate our objects using strings by using `ExpressibleByStringLiteral`. When using this protocol, make sure you implement at least the `public required init(stringLiteral:)` method:

```swift
public class Doll: ExpressibleByStringLiteral {
  var name: String
  var maker: String
  
  public required init(stringLiteral value: StringLiteralType) {
    let splat = value.split(separator: "|")
    self.name = String(splat.first ?? "")
    self.maker = String(splat.last ?? "")
  }
}
```

With this, we can instantiate a new `Doll` with a string with the format `DOLL_NAME|DOLL_MAKER`, as so:

```swift
let aliceDoll: Doll = "Classical Alice|Pullip"

print("\(aliceDoll.name) was made by \(aliceDoll.maker)") // Prints "Classical Alice was made by Pullip
```

This is one of my personal favorites, as it can help you create nice initializers for complex data.

## ExpressibleByIntegerLiteral and ExpressibleByFloatLiteral

These two are very similar, and as such they share the same section.

It is very easy to use a number to instantiate our objects. The following example declares `MultipliedNumber`, which takes a number and multiplies it by itself:

```swift
public class MultipliedNumber: ExpressibleByIntegerLiteral {
  let number: Int
  
  public required init(integerLiteral value: IntegerLiteralType) {
    self.number = value * value
  }
}
```

```swift
let myNumber: MultipliedNumber = 8

print("myNumber is \(myNumber.number)")
```

## ExpressibleByBooleanLiteral

I really like this one, because if you have an object that simply keeps track of different boolean states, you can use this to initialize them all to the same value.

```swift
public class DollFlags: ExpressibleByBooleanLiteral {
  var hasWig: Bool
  var hasStockOutfit: Bool
  var hasExtraAccessories: Bool
  
  public required init(booleanLiteral value: BooleanLiteralType) {
    self.hasWig = value
    self.hasStockOutfit = value
    self.hasExtraAccessories = value
  }
}
```

Now we can initialize them all to the same value by initializing it as so:

```swift
let flags: DollFlags = true
```

You can naturally do much more with it, but this is one of my favorite use cases.

## ExpressibleByArrayLiteral

Now we will see two of the most interesting ones due to their additional constraints. because Arrays and Dictionaries are typed in Swift, we need to keep that in mind when using `ExpressibleByArrayLiteral` and `ExpressibleByDictionaryLiteral`.

In the following example, we will create an object that takes an array of numbers, multiplies them by themselves, and stores that result:

```swift
public class ArrayNumberMultipler: ExpressibleByArrayLiteral {
  public typealias ArrayLiteralElement = Int
  
  let numbers: [ArrayLiteralElement]
  
  public required init(arrayLiteral elements: ArrayLiteralElement...) {
    self.numbers = elements.map { $0 * $0 }
  }
}
```

These protocols use associated types to assign the data type of the elements. In our case, our object can be initialized with an array of integers, so we assign `ArrayLiteralElement` to `Int`.

```Swif
let myNumbers: ArrayNumberMultipler = [2, 4, 6]

print(myNumbers.numbers) // Prints "[4, 16, 36]"
```

## ExpressibleByDictionaryLiteral

Finally, the last `ExpressibleBy-` protocol we will explore will allow us to instantiate objects with a dictionary. This can be very cool and handy in certain cases.

```swift
public class Doll: ExpressibleByDictionaryLiteral {
  public typealias Key = String
  public typealias Value = String
  
  let name: String?
  let maker: String?
  
  public required init(dictionaryLiteral elements: (Key, Value)...) {
    self.name = elements.filter { $0.0 == "name" }.first?.1 ?? ""
    self.maker = elements.filter { $0.0 == "maker" }.first?.1 ?? ""
  }
}
```

```swift
let doll: Doll = ["name": "Classical Alice", "maker": "Pullip"]
```

Once again, we have associated types, this time for the `Key` and `Value` of the dictionary.

# Conclusion

The `ExpressibleBy-` protocols are very helpful and they can help us write very expressive code. We shouldn't abuse them as they can be shocking for someone looking at a codebase the first time, but when used in moderation, they are one of my favorite features of Swift.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!