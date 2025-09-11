---
title: "Understanding Basic Data Structures in Swift: Dictionaries in Depth"
date: 2021-01-27T07:00:00-04:00
originalDate: 2021-01-24T22:11:35-04:00
publishDate: 2021-01-27T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - apple
 - dictionary
 - data structures
 - data
 - structures
 - foundation
categories:
 - development
description: "Get a deeper understanding of what dictionaries are and the theory behind them."
keywords:
 - swift
 - apple
 - dictionary
 - data structures
 - data
 - structures
 - foundation
---

Whether you are a seasoned developer with a lot of code out in the wild world, or you started learning programming this week, chances are you hace used (and seen) dictionaries being used in many places. Also known as *hashmaps* or *hash tables*, dictionaries allow us to store *key-value* mappings, from one object to another.

In this article we will study this structure which is known by everyone, and we will also learn about its quirks and unknown features.

# Dictionary Basics

Dictionaries map keys with values. Similar to an array, but instead of using numerical indexes, they use a *Hashable* as the key. For the vast majority of cases, this Hashable is a string. Each has is unique, and a dictionary cannot store the same key twice pointing to a different object. In fact, try running this somewhere:

```swift
let dollDic = [
  "Pullip": "Classical Alice",
  "Pullip": "Eileen"
]
```

Dictionary literals can do a lot of checking beforehand. You will see an error similar to the one below:

> Fatal error: Dictionary literal contains duplicate keys: file Swift/Dictionary.swift, line 826

When you attempt to set the same existing value at runtime, your code will not crash. Instead, it will replace the old one:

```swift
var dollDic = [
  "Pullip": "Classical Alice"
]

dollDic["Pullip"] = "Eileen"

print(dollDic["Pullip"]) // Eileen
```

## Keys Order

One question I see being asked very often is, why aren't the keys of a dictionary ordered when I print them or try to access them? Why are the keys in a different order every time I print an app?

There is nothing in the definition of a dictionary (or a hashmap) that guarantees an order for the keys. As part of the *hash* part, there is no way to make a guarantee of the order.

The *hash* is a calculated value. Dictionary keys get "converted" to a hash at runtime. Ultimately, dictionaries are *hash tables*, and that means that they need a very fast access of their index. Whatever you want to use as a key, is likely to be slow, so converting it to a hash ensures quick lookup in the table to retrieve it or store something new. Because keys are converted to hashes for lookup, the table does not keep track of the original keys\*. Instead, the keys may look like random data, but they are the result of an operation to make hash table lookup very quick.

*\*: While you can access the keys in Swift and they are not lost forever, the definition of a dictionary still does not guarantee any order. In Swift, you can access the keys by calling the `keys` property on a dictionary. This will return an array of the keys.*

## Keeping a dictionary order

With all that said, there may be times in which you do need to keep a certain order to your keys after all.

The first method is to call the `keys` property of a dictionary. This will return an array with the keys. this is an array you can sort and use that to access your dictionary.

```swift
var directory = [
  "e": "Eileen",
  "b": "Bloody Red Hood",
  "a": "Alice",
]

let keys = directory.keys.sorted()

keys.forEach { print(directory[$0]) }
```

Once you have your keys, you can sort them any way you like, and then just iterating over them and accessing the dictionary in that order will give you the results you want.

```
Alice
Bloody Red Hood
Eileen
```

## Introducing KeyValuePairs

But, there is a lesser known type in the Swift library called [`KeyValuePairs`](https://developer.apple.com/documentation/swift/keyvaluepairs). This object is a *key-value* store just like dictionaries, with the main difference being that the collection is ordered since the beginning, but another important difference is that this is not a *hash table*, so lookup is not as fast. While you are not likely to hit any constraints in most modern computing systems, do keep in mind that performance when using `KeyValuePairs` is worse.

The easiest way to initialize a `KeyValuePair` is to declare your variable of type `KeyValuePair` and then giving it a dictionary literal.

To be clear and avoid any confusion, the `KeyValuePair` will not accept anything and return it sorted for you. instead, it just means that it will keep the items in the order you gave them.

Also keep in mind that there is no guarantee that the keys will be unique. In fact this is perfectly valid:

```swift
var directory: KeyValuePairs = [
  "e": "Eileen",
  "b": "Bloody Red Hood",
  "a": "Alice",
  "b": "Betty"
]


directory.forEach { _, value in print(value) }
```

```
Eileen
Bloody Red Hood
Alice
Betty
```

Perhaps the reason this object isn't very well known is because ultimately, it's not really useful. You can get the same benefits just using a an array of `(String, String)` objects and prevent your codebase from getting polluted with lesser known objects.

# Hashable Implementation

While not very common, it's possible you may want to create your own keys for a dictionary.

`Hashable` is a protocol with very few requirements.

All you need to do is implement the `hashValue: Int` property.

The hash of an object has to be unique per run of the app. It doesn't even have to be the same across consistent launches. Generally when I need to create my own Hashables, I try my best to user an underlying properly to supply it.

```swift
struct Doll: Hashable {
  let name: String
  let maker: String
  
  var hashValue: Int {
    name.hashValue
  }
}

let alice = Doll(name: "Classical Alice", maker: "Pullip")
let delia = Doll(name: "Delia", maker: "Myou")

let dollCount = [alice: 2, delia: 3]
```

In the example above, I'm using the `name` property's `hashValue` as the hash for my object. Sometimes this will help you build more obvious code, but make sure you don't over-engineer on top of this.

# Conclusion

Dictionaries are very common, in all programming languages, all over the world. Understanding the gotchas is important, and the features Swift provides for them are worthwhile a bit more study.

