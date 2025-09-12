---
title: "Introduction to Apple's Unified Logging System on iOS 14 in Swift"
date: 2020-08-26T07:00:00-04:00
draft: false
originalDate: 2020-08-23T21:12:32-04:00
publishDate: 2020-08-26T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - watchos
 - macos
 - tvos
 - logging
 - oslog
categories:
 - development
description: "Learn about the Unified Logging system on Apple Platforms."
keywords:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - watchos
 - macos
 - tvos
 - logging
 - oslog
---


It is no surprise that software tend to write logs to a local file as they execute. As events, errors, or exceptional situations occur, a lot of software takes note of them using a local logging solution. This is done because these practices can allow us to troubleshoot problems for our users, find bugs, and in general understand the behavior of our software in untested or lesser tested scenarios.

When comes to iOS and other Apple platforms, there have always been third party dependencies that allow you to do this. A lot of developers roll their own solution and write events in plain text files. It wasn't until iOS 8 and macOS 10.10 that Apple provided us with a unified logging system that is easy to use and is very performant - [`OSLog`](https://developer.apple.com/documentation/os/oslog).

`OSLog` provided a lot of first party facilities for local logging, but it's not quite there.

This year, at WWDC2020, Apple showed us further improvements to their unified logging systems, providing a simple and consistent API that is easy to use, performant, and optimized for different logging scenarios.

In this series of articles we will explore these new unified logging APIs and how you can use them in your apps. Note that we will be focusing on the new WWDC2020 APIs. If you need to implement logging in earlier versions, you will need to leverage `OSLog` instead.

# Introducing Logging

Traditional logging frameworks will always write their events to a file. Apple's logging solution is very powerful, and it can persist events only in memory or to disk if necessary. There is default configurations to how the system stores logs, but you can change them to suit your needs, including logging absolutely everything to a file, or everything to memory (which wouldn't make much sense).

When logging to a file, this is not a plain text file. Instead, Apple has a logging format that is performant and it can do a lot of things for you, including removing sensitive data from logs (without losing filtering options), and more.

Details to view your logs will come at a later time. For now, be aware that Apple provides many ways to view them, including using the Mac's Console.app, the `log` command-line tool, and the Xcode debugging console. We also have the option to read logs programmatically via the older `OSLog` system.

## Writing Logs From Your Code

### Logging Practices

Before we get into the actual logging, let us establish some good practices you may use when you want to start logging events, including good places to place the logging code:

* At the beginning and end of functions of tasks. Non-trivial functions are a good place to place logging code. If you have more complex tasks composed of smaller function calls, they are also good candidates for logging.
* Any general events: When networking calls succeed or fail, when opening files, etc.
* When significant errors take place. Unexpected errors that can leave your app in a weird state and significant errors in general should be logged.
* Unusual code paths. If there's code paths that should barely happen, log them as well. This is great to find weird bugs or bugs that happen due to unusual user actions.
* Log after each line of a multi-step execution.

When logging messages, you can log more than just static strings. You can incorporate static strings, numbers, and other Objective-C objects into your logging. Even with this huge flexibility, the system will by default automatically remove sensitive information from your messages.

### Creating Logs

A log object centralizes all the logs that occur at a specific place of your app. All unified logging is done through this logging object. In Swift, you can use the `Logger` object introduced at WWDC. Objective-C folks will have to continue using `OSLog`.

Logging objects come with two basic filtering options. You can use them to filter out information as you try to diagnose issues in a specific place of your app. Because unified logging can produce an overwhelming amount of logs, you should make good use of these two options:

* The **Subsystem** is a functional area of your part. Multiple processes should be their own subsystems. You can even define a screen as a "screen" in your app. The subsystem can be anything that contextually makes sense in your app. I like to define a subsystem as an internal framework of my app. It is a good idea to use reverse DNS notation to name your subsystems.
* The **Category** can be used to define a component of your app inside a given subsystem. You may categorize your subsystem based on the UI that drives your interactions, the data processed by the component, or networking code related to that category. Once again, you have to define the category based on the context of your usage. Unlike the Subsystem, you are free to use any format for these strings, not just reverse DNS notation.

#### Log Levels

There are different log levels, and you should choose the right level for everything you need to log. Because different log levels store the logs differently by default, it is important to understand all the levels before you implement unified logging into your apps:

* **Debug**: Generally used to log "everything" when developing your app. Because debug logging can be aggressive, the logs are not stored on disk as they are only useful during development.
* **Info**: This level only stores logs on disk when collected with the `log` tool. Use it when you need to collect important but not essential information about your app.
* **Notice**: This is also the default level. Use it to capture information that may be important to diagnose issues, such as tasks that may cause errors. These are persisted to disk up to a storage limit.
* **Error**: Persisted to disk, up to a storage limit. Use this level to capture any error. Log information related to the error itself to make it easier to troubleshoot either.
* **Fault**:  Persisted to disk, up to a storage limit. Log situations that can leave your app in an invalid state or that may cause bugs. Essentially, use this to log harsher conditions than simple errors.

If you log messages with `log`, you can make it so `Info` messages are also stored on disk. The last three are compressed before being stored, so they are very efficient.

#### Creating Logs In Code.

With all that theory out of the way, we will explore the simplest use case for logging in this article - static strings. In a later article we will explore advanced logging with dynamic and stripped data, as it's a topic that deserves its own article.

To create a `Logger`, you can use the default initializer that takes no parameters to use the default subsystem. This is not recommended. You should use the `init(subsystem:category)` initializer instead. 

Don't forget to import `OSLog`, you need it regardless if you need Swift only or Objective-C support.

```
import OSLog

var logger = Logger(subsystem: "com.andyibanez.com.nae.profile", category: "Networking")
```

Finally, actually logging strings is really easy. The object has a method for each level.

```swift
logger.debug("Initializing networking object")

logger.info("Initialized networking object")

logger.notice("Networking object is currently nil")

logger.error("An error occured initializing network object")

logger.fault("Networking object has gone away prematurely")
```

And that's it for static strings! In subsequent articles in this series we will explore how to create dynamic logging and how to read the files created by this framework.

# Conclusion

Logging can help you diagnose issues that just don't occur at development time. It can also help you troubleshoot other issues your users may experience. The unified logging system is very performant and very easy to use for this task.
