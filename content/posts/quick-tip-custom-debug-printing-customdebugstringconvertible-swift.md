---
title: "Quick Tip: Custom Debug Printing with CustomDebugStringConvertible in Swift"
date: 2020-11-15T15:58:59-04:00
draft: false
publishDate: 2020-11-18T07:00:00-04:00
originalDate: 2020-11-15T15:58:59-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
categories:
 - development
description: "Learn how to print custom objects to the console."
keywords:
 - swift
 - programming
 - apple
---

Printing stuff to the console is a simple but powerful step we can take when debugging our apps. But there are times when we want to print an object and we actually get something entirely different, often also useless.

For example, this commonly happens when working with classes and printing instances of them.

```swift
class User {
  let id: Int
  let name: String
  
  init(id: Int, name: String) {
    self.id = id
    self.name = name
  }
}

let andy = User(id: 1, name: "Andy")

print(andy)
```

In a playground, this will print:

```text
__lldb_expr_6.User
```

This won't really help us find or observe our objects as they get printed to find issues in our app.

# Introducing CustomDebugStringConvertible

Luckily for us, Swift comes with the `CustomDebugStringConvertible` which we can adopt to print custom types. By using this protocol, we can make it so `print` prints more relevant info to the console when working with our custom objects.

To adopt it, you simply need to implement the `debugDescription` property. This property is of type String and you can return any string you want.

```swift
class User: CustomDebugStringConvertible {
  let id: Int
  let name: String
  
  init(id: Int, name: String) {
    self.id = id
    self.name = name
  }
  
  var debugDescription: String {
    return
    """
    -----------------------------------------------
    User ID: \(id)
    Name: \(name)
    -----------------------------------------------
    """
  }
}
```

Now, when we `print(andy)`, we will get this in the console:

```text
-----------------------------------------------
User ID: 1
Name: Andy
-----------------------------------------------
```

## Printing More Properties with Reflection and CustomDebugStringConvertible

If you have objects with many properties, it will take a good amount of effort to print all the properties of your object. To go around this, we can leverage [reflection](https://www.andyibanez.com/posts/quick-introduction-reflection-swift/) to quickly get and iterate over the properties of our object.

```swift
var debugDescription: String {
  let mirror = Mirror(reflecting: self)
  var debugString = "----------------"
  mirror.children.forEach {
    debugString += "\n\($0.label ?? ""): \($0.value ?? "")"
  }
  
  debugString += "\n----------------"
  return debugString
}
```

```text
----------------
id: 1
name: Andy
----------------
```

# Conclusion

While I still prefer breakpoints when it comes to debugging, getting proper prints is useful, especially if you want to observer the values of multiple variables, probably in different scopes, and don't want to deal with a breakpoint on each. Hopefully this also helps library developers implement "print-friendly" objects.

