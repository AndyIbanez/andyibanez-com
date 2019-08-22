---
title: "Introducing to Key-Value Coding for Apple's Platforms."
date: 2019-08-20T11:49:27-04:00
draft: true
aliases:
 - /key-value-coding-key-value-observing-cocoa-bindings-basic-tutorial/
---

*In my old website, this article was called "Key-Value Coding, Key-Value Observing, and Cocoa Bindings: A Basic Tutorial" and it was published all the way back in 2012. The examples were given in Objective-C. This updated article uses examples written in Swift, and it has been revised to be easier to understand.*

Key-Value Coding allows objects to provide indirect access to their properties (without calling their accessor directly). This allows us to do some neat tricks, like observing model objects and updating our UIs when they change. KVC coding has been possible on iOS for a long time, and macOS supports an additional related feature called Cocoa Bindings. In this article, we will explore Key-Value Coding, Key-Value Observing, and Cocoa Bindings, and we will learn how they can be applied to real-world apps.

But before we dive in deeper, we need to understand some terminology first. This will be useful when we start writing examples and seeing other people's codes:

* **Key-Value Coding (KVC)** Is the mechanism that allows you to observe objects and change them through a specific API. You make your objects Key-Value Coding Compliant by conforming to the `NSKeyValueCoding` protocol. As KVC started with Objective-C, you can expect to see it all over Apple's frameworks. In fact, the original `NSObject` complies with this protocol, so the vast majority of objects in Apple's frameworks support it.
* **Key-Value Observing (KVO)** is part of KVC and it's the most common task that you do with it. KVO allows you to observe object properties and react to their changes. You might be thinking why this is necessary in Swift, when we already have a native way of doing something similar (`didSet`), and the answer is that with `didSet`, your object reacts to changes to itself, whereas with KVO any object can react to any change in any other object.
* **Cocoa Bindings** is specific to macOS only. It cannot be used in iOS, iPadOS, watchOS, or TVOS. Cocoa Bindings allow you to bind a property with a GUI element, so when one changes, the other one changes as well. This two-directional communication helps macOS apps keep both data and interfaces up-to-date.