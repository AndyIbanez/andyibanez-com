---
title: "Quick Introduction Reflection in Swift"
date: 2020-05-06T22:24:46-04:00
originalDate: 2020-05-03T22:24:46-04:00
publishDate: 2020-05-06T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - dynamism
 - reflection
 - mirror
 - metaprogramming
description: ""
keywords:
 - swift
 - programming
 - apple
 - dynamism
 - reflection
 - mirror
 - metaprogramming
---

If you have been programming for a few years, you have undoubtedly come across the term *Reflection*. This feature allows us to inspect and work with the members of a type.

if this doesn't make sense, suppose you wanted to check *what* members a type has. How would you do this? Ideally you'd like to iterate over its members and print them. This is a very basic application of Reflection, but it should let think of other potential uses for it.

# Introducing Mirror

`Mirror` is an object that allows us to inspect the members of a type - it can be class, struct, or even a protocol. It's simplest use doesn't give much of a challenge.

You start by creating your object:

```swift
struct Person {
  let name: String
  let age: Int
}
```

Then create an object, and reflect it:

```swift
let andy = Person(name: "Andy Ibanez", age: 28)

let andyMirror = Mirror(reflecting: andy)

andyMirror.children.forEach {
  print("Member: \($0.label)")
  print("Value: \($0.value)")
}
```

The above code would print:

```
Member: Optional("name")
Value: Andy Ibanez
Member: Optional("age")
Value: 28
```

We iterate over each property, and print its value.

# Conclusion

Reflection is a very interesting feature that allows to create some sort of *meta-programming* in Swift. While not applicable to many use cases, it's important to be aware of its existence.

