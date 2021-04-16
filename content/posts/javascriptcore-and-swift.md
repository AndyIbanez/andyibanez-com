---
title: "JavaScriptCore and Swift"
date: 2021-04-14T07:00:00-04:00
originalDate: 2021-04-11T23:05:55-04:00
publishDate: 2021-04-14T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - apple
 - swift
 - xcode
 - javascript
 - programming
categories:
 - development
description: "Learn how interoperate between Swift and Javascript."
keywords:
 - apple
 - swift
 - xcode
 - javascript
 - programming
---

JavaScriptCore and Swift

Regardless how you feel about JavaScript as a programming language, there is one simple fact: JavaScript is pretty ubiquitous, and its uses have expanded beyond web scripting. It has become a pretty popular language for a vast array of domains. For this reason, making languages interoperate with it is pretty important, and both Swift and Objective-C are no exception. We can work with JavaScript, not only by executing JavaScript code directly from our Swift code, but we can even expose code from Swift and Objective-C to JavaScript. That's how important this language is, and these features open a world of possibilities.

JavaScriptCore will help us execute basic JavaScript code and export our Swift and Objective-C code to JavaScript. For the bunch of possibilities JavaScriptCore opens to us, it's actually a very simple framework with only a few symbols. In this article we will explore some common tasks we may want to perform with this framework.

## Executing JavaScript with JSContext

The most basic thing we can do with the framework is hand it over some JavaScript, evaluate it, and return its value. For this, we use `JSContext`.

```swift
let context = JSContext()
let sumValue = context?.evaluateScript("1 + 2 + 3")
if let sum = sumValue?.toInt32() {
    print("\(sum)")
}
```

`JSContext` is a JavaScript execution environment. It has some neat uses:

* Evaluate basic (or complex) JavaScript code from Swift or Objective-C.
* Make native Objective-C and Swift code available to JavaScript.

After calling `evaluateScript`, we will receive a `JSValue` object back. We can then use `JSValue` to pass data between JavaScript and Swift/JavaScript. In the example above, we performed a simple addition, converted the result to an `Int32`, and printed it.

`JSContext` has other interesting features as well. We can query the currently executing `JSContext` instance by calling the `current()` static method. We can even get the current callee by calling the `currentCallee` static method. Even more interesting we can get what "this" refers to by calling `JSContext.currentThis`. Finally, we can retrieve the current arguments by calling `JSContext.currentArguments`.

## JSVirtualMachine

All JavaScript code execution has a `JSVirtualMachine` somewhere behind this scenes. We can use this class directly when we need to support concurrent JavaScript execution and to manage memory when bridging between JavaScript and Swift/Objective-C.

Every `JSContext` is associated to a `JSVirtualMachine`, which you can get by calling the `virtualMachine` property, although one virtual machine can contain multiple contexts. Each Virtual Machine is its own world and its own environment, so while contexts within the same virtual machine can see each other, contexts belonging to different virtual machines are not aware of each other.

JavaScript is a concurrent affair. Any and all calls to JavaScript will be concurrent. If you need to execute JavaScript concurrently, simply create different `JSVirtualMachine` instances and execute them in different threads.

## Exporting Swift to JavaScript.

If we want our Swift (or Objective-C) objects to be available to JavaScript, we simply need to adopt the `JSExport` protocol. By adopting this protocol, we will be able to export our entire classes, instance methods, class methods, and properties to JavaScript. Many Foundation types automatically support this behavior, such as NSString.

Discussing JavaScript in-depth is not the topic of this article, so just be aware that in JavaScript, Object-Oriented Programming is supported through the use of Prototype Objects, and your own classes are exported to JavaScript as such.

```Swift
@objc class Doll: NSObject, JSExport {
    dynamic var name: String
    dynamic var maker: String
    dynamic var existence: Int
    
    init(name: String, maker: String, existence: Int) {
        self.name = name
        self.maker = maker
        self.existence = existence
    }
    
    let makeDoll: @convention(block) (String, String, Int) -> Doll = { name, maker, existence in
        return Doll(name: name, maker: maker, existence: existence)
    }
}

```

In order to export code to JavaScript, the first thing we need to keep in mind is that our objects must bridge to Objective-C - JavaScriptCore was introduced way before Swift was a thing. Second, we need to manually choose what properties and methods will be exported. In the case of properties, we need to provide a `@convention(block)` property that will bridge our code. This will export our functions with the right parameters and internally our Swift code will be called.

For properties, marking them as `dynamic` will ensure they are exported.

Finally, we can pass it to JavaScript by calling the `setObject(_:forKeyedSubscript)` method.

```
context?.setObject(Doll.self, forKeyedSubscript: "Doll")
```

# Conclusion

JavaScript is huge, and due to its huge community support you can find a lot of libraries for it. You may come to a point in which it may be easier to use JavaScript than Native Swift to solve a particular problem. It's also just a pretty cool feature in general that doesn't hurt to have in your toolbox.
