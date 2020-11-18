---
title: "Understanding KeyPaths in Swift"
date: 2020-11-11T07:00:00-04:00
publishDate: 2020-11-11T07:00:00-04:00
originalDate: 2020-11-09T11:35:08-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - keypaths
categories:
 - development
description: "Learn what KeyPaths are and how to use them in Swift."
keywords:
 - swift
 - programming
 - apple
 - keypaths
---

KeyPath. It sounds like a very fancy word. And it is a feature you have likely used it, either knowingly or unknowingly. KeyPaths are one of my favorite features in Swift, but they can be a bit tricky to understand. In this article we will explore what KeyPaths are, and when you may want to use them.

# Understanding KeyPaths

In simple words, a KeyPath is a reference to an actual property instead of a value.

## KeyPaths basics.

As we said above, KeyPath is a reference to a property instead of a value. If you can't wrap your head around this concepts, imagine a normal variable pointing to a normal value, such as:

```swift
let dollName = "Classical Alice"
```

The variable points to a *value*, and we can access that value anytime by just writing the variable name.

```swift
let completeDollName = "Pullip \(dollName)"
```

Everytime we write `dollName`, we are getting the *value* stored by that property.

Opposite to that, KeyPaths can hold a property itself.

KeyPaths point to properties in an object, so to show you how to create and use them, we will create a `Doll` object with three basic properties.

```swift
class Doll {
  let maker: String
  let name: String
  let releaseYear: Int
  
  init(name: String, maker: String, releaseYear: Int) {
    self.name = name
    self.maker = maker
    self.releaseYear = releaseYear
  }
}

let classAlice = Doll(name: "Classical Alice", maker: "Groove", releaseYear: 2013)
```

When we are referencing a KeyPath, we start with a backwards slash `\`, followed by the definition of the object (the class or struct name), followed by the property we want to "point" to.

```swift
let dollMaker = \Doll.maker

print(dollMaker)
```

`\Doll.maker` gives as a reference to the `maker` property of `Doll`. If you print `dollMaker`, Swift will print some weird meta info about the object, but that's OK - A KeyPath is a reference to a property! And you should not expect it to return a value directly.

By the time you do want to get a value when you have a KeyPath, the `keyPath` will help you do just that.

```swift
let dollMaker = \Doll.maker
let aliceMaker = classAlice[keyPath: dollMaker]
print(aliceMaker) // Pullip
```

When using KeyPaths within an object definition itself, you can omit the definition name, like this:

```swift
class Doll {
  //...
  func getMaker() -> String {
    return self[keyPath: \.maker]
  }
}
```

## The Usefulness of KeyPaths

I can hear some of you rumbling in the back, wondering why in the heck would this ever be useful to anyone. There's a few reasons for this.

First, KeyPaths are actually nothing new. They are used all over Apple's frameworks, and all Swift actually did was to provide a much better syntax for dealing with KeyPaths. Objective-C does support KeyPaths, but they are provided as strings when you need them, and this is error prone. In the beginning, Swift actually ported this mechanism directly, but through the process of its evolution, Swift arrive at this new syntax which won't let you introduce bugs based on simple typos when writing a KeyPath.

If you have played with SwiftUI, you have likely seen the `ForEach` and `List` views. These views expect an `Identifiable` object, but when you don't have them, you can provide a KeyPath to tell them what property to use to consider your data source as unique.

```swift
let dollArray = [classAlice, eileen, delia]
ForEach(dollArray, id: \.name) {
	// Do something with each doll
}
```

`ForEach` and `List` operate on unique data and they have internal optimizations to do their work. If our views are not `Identifiable`, we have a way to tell them to treat a property as unique. We do with KeyPaths. We are providing them with the `Doll.name` KeyPath and the views will assume the doll names are the unique property for each model. There is no other way to "explain" `ForEach` what property it should use to look for uniqueness.

KeyPaths are very powerful if you know how to use them.

Second, KeyPaths enable us to do what's called *metaprogramming*. Megaprogramming is a concept in which a program uses another program as its data. When it comes to KeyPaths, our apps themselves *are* their own data. This funky concept opens a lot of doors, and the best thing is, in Swift, it does in a very safe manner. Metaprogramming in Objective-C through KeyPaths is very possible, but as we said above, it doesn't have specialized `KeyPath` types, but rather it just uses KeyPaths as strings.

One more important thing before we move in, `KeyPaths` can reference properties nested deeply in other objects. Consider the following example:

```swift
class Maker {
  let name: String
  let fundedYear: Int
  let producedDolls: Int
  
  init(name: String, fundedYear: Int, producedDolls: Int) {
    self.name = name
    self.fundedYear = fundedYear
    self.producedDolls = producedDolls
  }
}

class Doll2 {
  let maker: Maker
  let name: String
  let releaseYear: Int
  
  init(name: String, maker: Maker, releaseYear: Int) {
    self.name = name
    self.maker = maker
    self.releaseYear = releaseYear
  }
}

let groove = Maker(name: "Groove", fundedYear: 2004, producedDolls: 100)
let alice2 = Doll2(name: "Classical Alice", maker: groove, releaseYear: 2013)
```

We have created a new `Maker` object, so each doll will reference the company that created them. 

Now, consider the following KeyPaths:

```swift
let dollMakerFundedYearKeyPath = \Doll2.maker.fundedYear
let makerFundedKeyPath = \Maker.fundedYear

let fundedYear = alice2[keyPath: dollMakerFundedYearKeyPath]
let fundedYear2 = alice2[keyPath: makerFundedKeyPath]
```

It's very important to keep in mind that `Doll2.maker.fundedYear`, and `\Maker.fundedYear` may look like they are pointing at the same property, they are not. These KeyPaths are entirely different, and they point to different things. Namely, the former points to a `maker.fundedYear` property, whereas the latter points simply to a `fundedYear`. KeyPaths are... Paths. Each period digs deeper in a property hierarchy, and it can become quite lengthy.

The neat thing about Swift is that the code above won't even compile. Because `Doll` doesn't have a `fundedYear` property, the compiler will catch the warning and it will stop you from introducing interesting bugs (looking at you, Objective-C).

## Passing KeyPaths Around

If you have tried to print a KeyPath directly, you have seen the console print something like this:

```swift
Swift.KeyPath<__lldb_expr_15.Doll2, Swift.Int>
Swift.KeyPath<__lldb_expr_15.Maker, Swift.Int>
Swift.KeyPath<__lldb_expr_15.Doll, Swift.String>
```

When dealing with KeyPaths, we have a literal object called `KeyPath` - There's a few more variations of KeyPaths, but we will explore them in a future article.

First we are going to explore an example that is not very useful in the real world, but it should help you get the hang of KeyPaths and why they are useful.

The following function takes a `Doll2`, and returns the value of any `String` property you want.

```swift
func getPropertyValue(in doll: Doll2, keyPath: KeyPath<Doll2, String>) -> String {
  return doll[keyPath: keyPath]
}

//...

let groove = Maker(name: "Groove", fundedYear: 2004, producedDolls: 100)
let alice2 = Doll2(name: "Classical Alice", maker: groove, releaseYear: 2013)

print(getPropertyValue(in: alice2, keyPath: \.name)) // Classical Alice
print(getPropertyValue(in: alice2, keyPath: \.maker.name)) // Groove
```

We can make this slightly more interesting, by adding generics and therefore being able to get the value of any property we want.

```swift
func getPropertyValue<Value>(in doll: Doll2, keyPath: KeyPath<Doll2, Value>) -> Value {
  return doll[keyPath: keyPath]
}

print(getPropertyValue(in: alice2, keyPath: \.releaseYear)) // 2013
print(getPropertyValue(in: alice2, keyPath: \.maker.fundedYear)) // 2004
```

Let's make a slightly more interesting example. While this example is not hard to recreate it with the usual high-order functions directly, I deliberately chose this one to show you how powerful KeyPaths are, and how dynamic the code you write can become.

```swift
extension Array where Element: Doll2 {
  func filtered<Value: Equatable>(by keyPath: KeyPath<Element, Value>, value: Value) -> [Element] {
    return Array(filter { $0[keyPath: keyPath] == value })
  }
}
```

This looks like a mouthful. A quick explanation is in order.

We are creating an extension that will apply to `Array` objects that contain `Doll2` objects. This extension has a `filtered(by:value)` function. This function will filter an array based on a KeyPath and its value, so if we want to keep all the dolls whose maker is `Groove`, it can do that. We use a generic called `Value` that conforms to equatable - in other words, values that can be compared, such as Strings, Ints, or anything else that can use `==` to compare amongst two instances of their own kind. Within the function we grab the KeyPath and compare it to the expected value.

When we want to filter `Doll2` arrays, we can call this as so:

```swift
let grooveMaker = Maker(name: "Groove", fundedYear: 2004, producedDolls: 100)
let myouMaker = Maker(name: "Myou", fundedYear: 2012, producedDolls: 50)

let classAlice = Doll2(name: "Classical Alice", maker: grooveMaker, releaseYear: 2013)
let eileen = Doll2(name: "Eileen", maker: grooveMaker, releaseYear: 2018)
let delia = Doll2(name: "Delia", maker: myouMaker, releaseYear: 2016)

let dolls = [classAlice, eileen, delia]

let onlyGrooveDolls = dolls.filtered(by: \.maker.name, value: "Groove")

onlyGrooveDolls.forEach { print($0.name) }
// Prints:
// Classical Alice
// Eileen
```

# Conclusion

KeyPaths open a lot of doors for us. By leveraging metaprogramming, we can write highly dynamic code. With Swift, this code is dynamic without sacrificing safety thanks to the fact we have literal `KeyPath` objects. KeyPaths also exist in Objective-C, but they aren't as safe.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!