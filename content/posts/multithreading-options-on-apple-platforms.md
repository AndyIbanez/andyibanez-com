---
title: "Multithreading Options on Apple Platforms"
date: 2021-02-24T07:00:00-04:00
originalDate: 2021-02-20T18:00:58-04:00
publishDate: 2021-02-24T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - macos
 - tvos
 - watchos
 - ipados
 - multithreading
 - nsoperation
 - gcd
 - grand central dispatch
categories:
 - development
description: "Meet the multithreading tools offered by Apple on all their platforms, and learn to choose the right one for your needs."
keywords:
 - swift
 - programming
 - apple
 - ios
 - macos
 - tvos
 - watchos
 - ipados
 - multithreading
 - nsoperation
 - gcd
 - grand central dispatch
---

We have reached the point in which computers are really fast. Especially Apple's, as they have control of both the hardware and software, so, oftentimes, some tasks that could be sped up with multithreading, are not necessary anymore. But, for those cases when you do need multithreading, we have many options available.

On Apple's platforms there is a surprising amount of concurrency tools. You are likely familiar with the most used one, the Grand Central Dispatch, `DispatchQueue`, which is pretty good and it covers the vast majority of use cases. But there are some tasks that can be done easier with other tools.

In this article, we will explore and introduce the concurrency options that we can use:

* The `NSOperationQueue` APIs
* Grand Central Dispatch (GCD)
* NSThreads
* pthreads

Along with a quick discussion on when you may prefer to use a tool over another tool.

I have sorted the tools by their "level" in terms of high-level or low-level APIs, where the higher level ones show up first.

This article assumes you are familiar with concurrency and why it's important in the context of iOS, macOS, watchOS, and tvOS apps.

# The NSOperationQueue APIs

NSOperation and its related APIs are actually one of my favorite set of tools we have for concurrency, not only on Apple's platforms, but overall.

These APIs are very high level, and as such they offer us some features that don't exist, or are harder to implement in other concurrency technologies.

NSOperationQueue and co. allow us to easily set tasks that depend on others, so you can easily start a concurrent task after another concurrent task has finished. It also allows you easily cancel other tasks.

You submit operations to a `NSOperationQueue` in the form of blocks, so it even offers really high level syntax.

I won't be talking much about these APIs here - despite the fact that I *love* `NSOperationQueue` -, because I have dedicated a whole article to it which you can find here: [Exploring the NSOperation APIs for Apple's Platforms](https://www.andyibanez.com/posts/exploring-the-nsoperation-apis/)

In general, I like using these APIs when I have to setup dependencies between tasks and I want an easy way to cancel them. Think of an app that can batch-download pictures and apply a filter to them without freezing your main thread.

# The Grand Central Dispatch

Whether you know the Grand Central Dispatch (we will simply call it GCD from now on) by its name or not, you have undoubtedly used it in many of your apps, especially if you used Apple-provided APIs that return their result in a different thread (such as `URLSession`). Does this line look familiar to you?

```
DispatchQueue.main.async {
  // Do something on the main thread
}
```

I'd be familiar if you never saw that particular line of code before.

The GCD is a set of concurrency APIs, developed by Apple themselves, which make it easy to support multi-core processors and other types of symmetric processing systems.

I want you to pay particular attention to that definition, because the GCD was built for multi-core processors, not just Apple's, and this is where one of its biggest advantages come into play. The GCD is actually [open source](https://apple.github.io/swift-corelibs-libdispatch/), so if you learn to use it on Apple's technologies, you are likely to find implementations of it on other entirely different platforms of all shapes and sizes, as long as they can run C, C++, Objective-C, or Swift.

The GCD is also a great little tool to make quick concurrent tasks easily, quickly, and without writing any ugly code. One of the most common uses for it among Apple platform developers is dispatching UI-updating tasks into the main thread after using asynchronous APIs such as `URLSession`.

One other common use is to delay the current execution of the thread for a definitive amount of time, which can sometimes be useful. In Swift in particular, this has a very nice syntax.

```swift
DispatchQueue.main.asyncAfter(deadline: .now() + .seconds(5)) {
  // This will be executed in the main thread after five seconds.
}
```

The block above will execute some code in the `.main` thread after five seconds.

Another great feature of the GCD is that it allows us to specify the priority of each task when submitting them into a queue. This way we can help the queue execute important task fasters by providing a `QOS` (quality of service) parameter.

```swift
DispatchQueue.global(qos: .userInteractive).async {
  // Any user-initiatied or user-interactive task has higher priority.
}

DispatchQueue.global(qos: .utility).async {
  // Tasks with uility QoS have lower priority.
}
```

The GCD provide us with a ready to use global queue where we can submit tasks to (and yes, you can create your own queues). There are multiple QoS flags you can provide depending on the perceived priority of the task:

```swift
@available(macOS 10.10, iOS 8.0, *)
case background

@available(macOS 10.10, iOS 8.0, *)
case utility

@available(macOS 10.10, iOS 8.0, *)
case `default`

@available(macOS 10.10, iOS 8.0, *)
case userInitiated

@available(macOS 10.10, iOS 8.0, *)
case userInteractive

case unspecified
```

In the future, I may write a dedicated article to the GCD. In the meantime, keep in mind the GCD is great to use when:

1. You need a quick way to "switch between" threads.
2. You need to specify priorities in your tasks and you want the system to prioritize them depending on a QoS flag.
3. You are potentially interested in using this amazing multithreading technology outside of Apple provided technologies.


# NSThreads and pthreads

I'm lumping these two together because they are both really low level, they have much more flexibility, and because their application is very similar: When you need direct control over thread creation and concurrency.

The GCD and NSOperation APIs have one thing in common: They handle all the actual thread management for you. When you are working with the GCD and NSOperation APIs, the system will choose when to create threads, when to destroy, when to dispatch them, and even where to dispatch them - see, NSOperation is particularly smart, and it won't actually do multithreading if it finds itself in a situation where not leaving the current thread is fine.

Sometimes though, computer smartness ain't actually smart, and you may find yourself in situations (although *very rarely*) in which you just aren't getting the behavior you want with the GCD or NSOperation\*. For those cases, we can manually take control of threading with NSThreads and pthreads.

pthreads are the lowest-level APIs. They are POSIX threads, meaning you can create and use pthread in any POSIX system. They are implemented in pure C. I won't be mentioning them much do their extremely niche use cases, but be aware they exist, and [have the documentation](https://computing.llnl.gov/tutorials/pthreads/) shall you be the one in a million dev who needs to drop down to such levels.

NSThreads, on the other hand, are bit more useful to most developers, even though they will not be touched by the vast majority of them. I think NSThreads have one particular advantage, and that is that the interface is very similar to Java threads. If you have worked with threads in Java, NSThread has a similar enough interface to make it easy to move among both languages. NSThread is a Foundation object, so it's good to keep that in mind.

Use NSThread when you need to have full control over concurrency, including thread creation and management (and as long as you are going to be responsible with your resource usage).

[NSThread docs](https://developer.apple.com/documentation/foundation/nsthread)

# Conclusion

Multithreading is complicated and to this day it is a hard problem to solve. Apple provides us with many options to deal with multithreading. NSOperation is great and high level, but GCD gets the job done very often. Balancing your options is important to become a better iOS developer.
