---
title: "Exploring the NSOperation APIs for Apple's Platforms"
date: 2019-08-21T14:47:38-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
aliases:
 - /multithreading-ios-mac-os-x-using-nsoperations/
tags:
 - swift
 - programming
 - apple
 - ios
 - macos
 - tvos
 - watchos
categories:
 - development
description: "Learn multithreading on Apple Platforms with the help of NSOperation, a high-level concurrency API."
keywords:
 - concurrency
 - ios
 - tvos
 - ipados
 - watchos
---

*The original title for this article was posted on my old website in 2012 and it was titled "Multithreading on iOS And Mac OS X Using NSOperations". The original examples were written in Objective-C. This article has been rewritten from scratch not only to give the examples in Swift, but also to improve the quality of the old article. It has been shortened, and both language and tone have been revised.*

Writing concurrent code is important and something that is done for almost non-trivial apps. If you want your app to handle expensive tasks without causing a bad experience for users (freezing the main thread), you will eventually have to deal with concurrency. And the good news is, if your concurrent task is complex enough, you can simplify how it's done with the help of an API that has been there for a very long time, but is ignored by many developers of Apple platforms: NSOperations.

Concurrency on iOS and macOS is nothing new. The technologies have also been moved to TVOS and watchOS. In fact, there's four concurrency methods that you can do in all these platforms:

* pthreads
* NSThread
* Grand Central Dispatch (GCD)
* NSOperation

Sorted from lower level to higher level, the vast majority of Apple developers will never touch pthreads (POSIX threads) or NSThreads, (though they can drop down to those levels if they need to for extremely sensitive performance applications), and most of them use the GCD in their day-to-day jobs. The GCD works fine for the vast majority of concurrent apps, and many developers will not need the extra functionalities offered by NSOperations.

But, while the GCD provides a great concurrency API for developers, sometimes your tasks may not scale well with it. The power of the NSOperation APIs is that they provide a high level interface for concurrency, and it allows you to set dependent tasks of each other very easily. Suppose you want to write an app that downloads photos from a Flickr feed, and after downloading them you want to apply a black and white filter on them. You can achieve this while creating a Download operation and a Filtering operation that depends on the Download operation.

In this article we will explore the NSOperations API and we will write some very basic code to show how they work, how to create dependent operations, and more.

# NSOperation Classes

The NSOperations API has a few classes that make it easy to interact with it.

The `NSOperation` class represents a task that you want to do concurrently. The code you want to perform concurrently is encapsulated within a `NSOperation` subclass. NSOperation itself is considered an abstract class, and as such you never use it directly (technically there isn't the concept of an abstract class in either Swift or Objective-C, but the [documentation](https://developer.apple.com/documentation/foundation/operation) uses that word when describing the API), but it provides a few subclasses that you can use instead.

The two subclasses Apple provides are:

* `NSInvocationOperation`: When using this class, you use the familiar target-action pattern to define what object should call what method as a concurrency task. You can use this subclass when you have an object with a method that you want to use in both concurrent threads and in your main thread. It also gives you a lot of flexibility thanks to the underlying dynamic Objective-C dispatching. It's very important to note that you can't use this one in Swift. The underlying class needed by this subclass to work (`NSInvocation`) is not available to be used in Swift. You can find more info about this [here](https://stackoverflow.com/a/26644944/648767), so in this article we will not cover this subclass.
* `NSBlockOperation`: You can use this operation to execute various closures at once. An `NSBlockOperation` is considered to have finished running when all its blocks have completed their execution.

You can also write your own NSOperation subclasses, but doing so is very complicated, unusual, and often unnecessary. The [documentation](https://developer.apple.com/documentation/foundation/operation) has some helpful subclassing notes, but chances are you will never have to do this. In this article, we will not explore how to do that as it is an extremely niché case.


All `NSOperation` subclasses support the same handy features:

* You can use dependencies to tell an operation to only execute once another operation has finished running.
* You can monitor their state through the use of Key-Value Observing.
* You can cancel operations at any time, whether they are running or have yet to run. You can implement your custom cancellation events if you need to.

When you have an `NSOperation` subclass ready, you will normally want to hand it over to an `NSOperationQueue`. You can actually choose to start each operation directly by calling their `start()` method, but doing so does not guarantee that the operation will run in a different thread. It could choose to run in your main thread. Using the queue is the only way to ensure your tasks are actually concurrent. All operations have a method called `isConcurrent` which you can use to check if they are running in a background thread or in your main thread.

You can query the status of your operations by some properties. All operations have the following:

* `isConcurrent`. You can use this one to see if the operation is running in a different thread than where it was called or not.
* `isExecuting`
* `isFinished`
* `isCancelled`

# Exploring the API

With all that theory out of the way, it's time to get our feet wet. We will be running these examples in a simple Command Line project in Xcode.

## NSBlockOperation

Using `NSBlockOperation` is very easy. In the example below, we will create two different `NSBlockOperation`. One will count from 1 to 10, and the other one will count from 1 to 20. You will see that there's no consistent order in how the numbers are printed, despite the fact that we queue the `from1to10`operation before `from11to20`.

```swift
class NumberCounter: NSObject {
    func startCounting() {
        
        /// We will need a queue for this.
        let operationQueue = OperationQueue()
        
        /// You can give your queue an optional name, if you need to identify it later.
        operationQueue.name = "Counting queue"
        
        /// This will just count from 1 to 10...
        let from1To10 = BlockOperation {
            for i in (1 ... 10) {
                print(i)
            }
        }
        
        /// ... and this from 11 to 20
        let from11To20 = BlockOperation {
            for i in (11 ... 20) {
                print(i)
            }
        }
        
        /// Add the operations to the queue
        operationQueue.addOperation(from1To10)
        operationQueue.addOperation(from11To20)
        
        /// To ensure the program doesn't exit early while the operations are running.
        operationQueue.waitUntilAllOperationsAreFinished()
    }
}

let counter = NumberCounter()
counter.startCounting()
```

Run this sample of code a couple of times. You will see that each time, the order of the numbers is different.

To fix this, we will make `from1to10` be a dependency of `from11To20`. When we do this, the numbers will be printed in the order you expect, no matter how many times you run the program. Add this code before you add the operations to the queue:

```Swift
 /// from11To20 should depend on from1To10, so the numbers are added in the right order.
 from11To20.addDependency(from1To10)
```

And just like that, the numbers will be printed in the right order, since `from11To20` will not run unless `from1To10` has completed its execution.

### A Tiny Trick

As I was writing this, I noticed that you can directly add a block to a `NSOperationQueue` without having to wrap it in a `NSBlockOperation`. It looks like this:

```swift
operationQueue.addOperation {
    for i in (21 ... 31) {
        print(i)
    }
}
```

This makes simpler concurrency easier to do in some cases. Just keep in mind that since you are passing a block directly, you do not have the same benefits as `NSBlockOperation`. For example, you will not be able to cancel this operation early, and you cannot query it's state with its properties.

# Conclusion

The `NSOperation` API is really good to do complex concurrently easily. You have a lot of control over your operations and managing dependencies is very easy.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!