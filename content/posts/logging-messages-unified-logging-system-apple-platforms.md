---
title: "Logging Messages With the Unified Logging System on Apple Platforms"
date: 2020-09-09T07:00:00-04:00
draft: false
originalDate: 2020-09-07T09:42:18-04:00
publishDate: 2020-09-09T07:00:00-04:00
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
description: "Learn how to format and strip sensitive data when logging with OSLog."
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

Last time we [talked about the basics of the Unified Logging System](https://www.andyibanez.com/posts/introduction-apples-unified-logging-system-ios14-swift/), we set the basic concepts and code we need to write logs, along with the different logging levels, and more.

In this article we will talk about actually logging messages, how the framework is "smart enough" to strip out sensitive user info by default, and how we can control what gets stripped.

# Logging Messages

The framework supports interpolated strings right out of the box when you are using the new system in Swift.

```swift
let elementCount = (1...3).count
let username = "Andy"
         
logger.notice("The array contains \(elementCount) elements")
logger.debug("Logged in username \(username)")
```

It's good to know that you can do this by default, but it's more interesting how there are some formatters ready to be used.

## Formatting Logging Variables

You can format your variables in the following ways:

* Specify the width of the variable and align its contents within it.
* Format integers as decimal, hex, or even octal numbers.
* Format floating-point numbers used fixed-point, exponential, or hybrid notation.
* Format booleans as true/false or yes/no strings.
* Setting the precision for floating point numbers.
* Setting the minimum number of digits.
* Specifying if a number needs an explicit plus or minus sign.\

To set these options, your string interpolations can take additional parameters for their formatting, such as:

```swift
let shouldPromptRating = true
logger.notice("Should prompt rating on completion: \(shouldPromptRating, format: .answer)")

let aFloat = 23434.29003493
logger.notice("Distance is \(aFloat, format: .exponential(precision: 3))")
```

Mentioning all the different formatting options and how to do them is beyond the scope of this article. As long as you are aware they exist, this section of the article has done its job. There is a very complete section on formatting in the [official docs](https://developer.apple.com/documentation/os/logging/generating_log_messages_from_your_code).

## Redacting Sensitive Info From Logging Messages.

It is recommended that you keep your logging to static strings and numbers when you log, but sometimes, you just really need to know about what kind of specific dynamic input data is causing troubles for your users.

The system will redact the contents of dynamic strings and other complex dynamic objects. If you need to explicitly log the content of dynamic strings, you can specify the `privacy` parameter when you perform string interpolation.

```swift
logger.notice("User logged in with username \(username, privacy: .public)")
```

Integer, floating-point, and booleans are not redacted by default. But you can manually redact them if you deem it necessary, using the same method as un-redacting strings.

```swift
let applesBought = 5
logger.notice("User bought \(applesBought, privacy: .private) apples")
```

There is one more thing related privacy. Suppose you need to login unique user IDs, but doing so would be invasive for user privacy. It may be important to know that a specific user ID us having problems, so how can you view this ID without seeing the ID itself?

The `.private` enum has a `.mask` initializer you can use. It will create a unique hash for the same input, but you will never see the real value of the input.

```swift
logger.notice("User \(userId, privacy: .private(mask: .hash)) logged in")
```

We can now reference all the logs with this specific user, without leaking any information about them.

# Conclusion

The unified logging system has all the facilities to format logs in a way that make sense in the context of your app, and they have all the tools you need to strip out potentially personal information, without losing references to it and without undermining the usefulness of the logging system.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.