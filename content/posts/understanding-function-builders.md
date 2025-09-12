---
title: "Understanding Function Builders in Swift"
date: 2020-03-11T07:00:00-04:00
originalDate: 2020-02-28T16:49:57-04:00
draft: false
publishDate: 2020-03-11T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - function builders
 - wwdc2019
categories:
 - development
description: "Learn what Function Builders in Swift are, and how to use them"
keywords:
 - swift
 - programming
 - apple
 - function builders
 - wwdc2019
---

WWDC2020 is just around the corner\*, and it hasn't been one year since WWDC2019 took place. There is still a lot of ground to cover regarding the new tools and APIs demonstrated then. and In this article we will focus on a feature new to Swift itself: Function Builders.

*\*: Maybe. :(*

If you have been hacking away at SwiftUI, you have probably been wondering how it makes it possible to build great UIs with very nice syntactic sugar. Other than [property wrappers](https://www.andyibanez.com/posts/understanding-property-wrappers-swift/), SwiftUI is also possible thanks to Function Builders. In this article, we will briefly mention how SwiftUI uses Function Builders, and later we will create our own function builders that have nothing not do with SwiftUI. This way, it will become evident why Function Builders are really neat, and why they don't have to be strictly tied to SwiftUI.

# Understanding Function Builders

"Function Builder" is a very fancy name for something very simple. Function Builders are nothing more and nothing less than syntactic sugar. With function builders, you can create more natural syntax for your code to make it more intuitive and easier to use and understand.

Function builders make it easier to build complex objects by providing more natural syntax for their creation.

If you have played with SwiftUI, you have most likely used `HStack` and `VStack` objects. They are *objects* though, so how come we can do something like this?:

```swift
HStack {
    Text("Hey")
    Text("I'm Andy")
}
```

The answer is that part of `HStack`'s constructor takes a `ViewBuilder` called `content`:

```swift
init(alignment: VerticalAlignment = .center, spacing: CGFloat? = nil, @ViewBuilder content: () -> Content)
```

So when we use the `content` parameter, we are instructing SwiftUI to create a `HStack` composed of multiple different views within - a complex view. SwiftUI will later convert it into in an object behind the scenes (a `TupleView<View, View>`, to be more specific) that it uses internally in order to render it. You can theoretically use SwiftUI without the syntactic sugar provided by Function Builders, but your code would look like a soup of generics in a world with no horizontal limits.

SwiftUI has multiple objects that work like `HStack`. Wouldn't it be interesting if we could create our own function builders? To better understand Function Builders and have a better idea of what SwiftUI does behind the scenes with them, we are going to build a few them to get the concept across.

## Building Custom Function Builders

To create a function builder, you need to declare a `struct` marked as `@_functionBuilder`. Next, your struct must have a static method called `buildBlock`, and this method must take at least one argument of any type. This argument will be what you write inside the curly braces when calling your function builder. In SwiftUI, these are `View`s. Finally, the `buildBlock` function can return something to the caller, which can be any type as well.

### A Simple Function Builder

In the following example, we are going to create a function builder that takes strings, and returns their lengths. We will later explore how to use it:

```swift
@_functionBuilder
struct StringCharacterCounterBuilder {
  static func buildBlock(_ strings: String...) -> [Int] {
    return strings.map { $0.count }
  }
}
```

### Utilizing Function Builders

Now that we have the function builder, the only thing left to do is to use it.

To do that, we will create a class with an initializer that takes an argument marked with our function builder:

```swift
class CharacterCounter {
  let counterArray: [Int]
  
  init(@StringCharacterCounterBuilder _ content: () -> [Int]) {
    counterArray = content()
  }
  
  func showCounts() {
    counterArray.forEach { print($0) }
  }
}
```

And finally, we can call our initializer like this:\

```swift
let characterCounts = CharacterCounter {
  "Andy"
  "Ibanez"
  "Collects Pullip"
}
```

Call the `showCounts()` to see it contains the content you expect:

```swift
characterCounts.showCounts()
```

```
4
6
15
```

Now, I realize this is a very abstract example, but hopefully you can see how function builders are used, and how they work.

In short, Function Builders:

1. Let you create syntactic sugar to make code easier to understand and make it more intuitive.
2. *Transform* content and hand it to the caller.
3. Allows us to create neat Domain Specific Languages (*DSL*), which is what SwiftUI essentially is.


The above example was contrived, but if you understood how it works, let's try building something more interesting.

# Building a Real Function Builder

The previous example was extremely generic and its main purpose was to show *how* function builders work and the syntax around them. In this section we will explore a real function builder you may want to use in your own projects. We will create function builders to create `UIAlertController`s declaratively.

## Easier to Use UIAlertControllers With Function Builders

If I had to give an opinion on `UIAlertController`, is that it is one of the most tedious APIs to use on iOS development. First, you have to create your alert controller, and then you have to create each action item separately, just to add each one of them, one by one, to the alert controller manually. You can make it easier by extending `UIAlertController` to add a variable amount of actions, but we can make it even easier (and more interesting), if we create a small DSL *just* to create `UIAlertController`s with actions.

Our end goal will allow us to create `UIAlertController`s writing code like this:

```swift
let alert = AlertController(
    title: "Delete all data?",
    message: "All your data will be deleted!",
    style: .alert) {
        
        AlertAction {
            DestructiveTitle("Yes, Delete it All")
            AlertHandler {
                print("Deleting all data")
            }
        }
        
        AlertAction {
            DefaultTitle("Show More Options")
            AlertHandler {
                print("showing more options")
            }
        }
        
        AlertAction {
            CancelTitle("No, Don't Delete Anything")
            AlertHandler()
        }
}

present(alert.alertController, animated: true)
```

![UIAlertController created with Function Builders](/img/fbs_alertcontroller.png)

In contrast, building this same `UIAlertController` in pure UIKit would be longer than it needs to be:

```swift
let alert = UIAlertController(
    title: "Delete all data?",
    message: "All your data will be deleted!",
    preferredStyle: .alert)

let deleteAction = UIAlertAction(title: "Yes, Delete it All", style: .destructive) { (_) in
    print("Deleting all data")
}

let moreOptionsAction = UIAlertAction(title: "Show More Options", style: .default) { (_) in
    print("Show more options")
}

let cancelAction = UIAlertAction(title: "No, Don't Delete Anything", style: .cancel, handler: nil)

alert.addAction(deleteAction)
alert.addAction(moreOptionsAction)
alert.addAction(cancelAction)

present(alert, animated: true)
```

The former is a more declarative approach, and it's both much quicker and easier to type.

### Supporting Classes

We will start by defining a few basic classes. These classes cover the title styles for the alerts, and the action to execute when one of them is tapped.

This small protocol definition will be used for the buttons:

```swift
protocol AlertActionStyleProtocol {
    var title: String { get }
    var style: UIAlertAction.Style { get }
}
```

```swift
struct DefaultTitle: AlertActionStyleProtocol {
    let title: String
    let style: UIAlertAction.Style
    
    init(_ title: String) {
        self.title = title
        self.style = .default
    }
}

struct CancelTitle: AlertActionStyleProtocol {
    let title: String
    let style: UIAlertAction.Style
    
    init(_ title: String) {
        self.title = title
        self.style = .cancel
    }
}

struct DestructiveTitle: AlertActionStyleProtocol {
    let title: String
    let style: UIAlertAction.Style
    
    init(_ title: String) {
        self.title = title
        self.style = .destructive
    }
}
```

One thing to keep in mind is that, when we write declarative code, we sometimes have to write more code in order to make the *end result* easier to write. You can probably make this better using inheritance, but I don't think there's a much of a problem doing it this way here.

The last independent class we need is the one that will handle the alert:

```swift
struct AlertHandler {
    let action: () -> Void
    init(_ action: @escaping () -> Void = {}) {
        self.action = action
    }
}
```

Now we can start writing the function builders.

### The Function Builders

The first function builder we will build will allow us to create the `UIAlertAction`s.

```swift
@_functionBuilder
struct UIAlertActionBuilder {
    static func buildBlock
        (
        _ style: AlertActionStyleProtocol,
        _ alertHandler: AlertHandler
    ) -> UIAlertAction {
        return UIAlertAction(title: style.title, style: style.style) { _ in
            alertHandler.action()
        }
    }
}
```

The object that will make use of this function builder is the following:


```swift
class AlertAction {
    let alertAction: UIAlertAction
    
    init(@UIAlertActionBuilder _ content: () -> UIAlertAction) {
        alertAction = content()
    }
}
```

So far, all the code we have written allows us to create `UIAlertAction`s with this new syntax:

```swift
AlertAction {
    DestructiveTitle("Yes, Delete it All")
    AlertHandler {
        print("Deleting all data")
    }
}
```

The body of the `AlertAction` calls the function builder. You can think of each statement as a parameter to the function builder.

The last function builder will allow us to create `UIAlertAction`s from `UIAlertAction`s, an we will use that to create the final `AlertController`:

```swift
struct AlertControllerBuilder {
    static func buildBlock(_ actions: AlertAction...) -> [UIAlertAction] {
        return actions.map { $0.alertAction }
    }
}
```

Finally, the class that will make use of the `AlertControllerBuilder` will create the `UIAlertController`:

```swift
class AlertController {
    let alertController: UIAlertController
    
    init
        (
        title: String,
        message: String,
        style: UIAlertController.Style,
        @AlertControllerBuilder _ content: () -> [UIAlertAction]
    ) {
        let alert = UIAlertController(
            title: title,
            message: message,
            preferredStyle: style
        )
        
        content().forEach { alert.addAction($0) }
        
        self.alertController = alert
    }
}
```

You can now use the code in the top of the session to create `UIAlertController`s much easier, faster, and with nice declarative syntax.

I will be curious to see if you have any ideas to improve this. If you play around with this idea and come up with any interesting improvements, please let me know.

# Conclusion

Swift 5 introduced many fascinating features to Swift. From property wrappers to function builders. If you use them responsibly, you can create DSLs for many tasks no matter how specific, and simplify the use of common APIs along the way.

I have converted the `AlertController` into a Swift Package you can add to your own projects. Its name is `DeclarativeAlertController`, and you can find it [here](https://github.com/AndyIbanez/DeclarativeAlertController).

