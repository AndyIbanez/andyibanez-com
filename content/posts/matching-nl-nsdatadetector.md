---
title: "Matching Natural Language Text for Predefined Data Patterns on Apple's Devices"
date: 2020-02-05T07:00:00-04:00
originalDate: 2020-02-02T12:21:23-04:00
publishDate: 2020-02-05T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - tvos
 - watchos
 - macos
 - natural language
categories:
 - development
description: "Learn to use the NSDataDetector class to match patterns in natural language."
keywords:
 - swift
 - ios
 - ipados
 - programming
 - tvos
 - watchos
 - macos
 - natural language
---

iOS has a lot of APIs that deal with natural language detection. One such class is [`NSDataDetector`](https://developer.apple.com/documentation/foundation/nsdatadetector). This class allows you to match different kinds of data in text, including dates, time, links, and more. This class, actually introduced a very long time ago (in the iOS 4.0 days!) makes it very easy to find this kind of data in strings. In this article we will explore how to use this very old class - whose documentation is Objective-C only at this time - in Swift, and how to do common tasks with it.

This class is a subclass of [`NSRegularExpression`](https://developer.apple.com/documentation/foundation/nsregularexpression), so you can expect it to not be entirely perfect with some data. For example, my country's addresses do not follow any kind of format used in developed countries, so address detection fails for me most of the time.

That said you can expect it to work with most data. Phone numbers, URLs, and dates/times are likely to be recognized without an issue.

# Finding Patterns in Natural Language.

When you create an instance of this class, you specify what kind of content you want to find. A checking type is a [`NSTextCheckingResult.CheckingType`](https://developer.apple.com/documentation/foundation/nstextcheckingresult/checkingtype) object, and you can specify more than one at a time.

In the below example, we will create a data detector that can find links and dates in a string:

```swift
let detector = try! NSDataDetector(
  types: NSTextCheckingResult.CheckingType.link.rawValue | NSTextCheckingResult.CheckingType.date.rawValue
)
```

As we said earlier, this is an Objective-C API that comes from the early iOS days, so it may look a little bit weird (and ugly), but it works.

We need to use the `rawValue`s of the types, and we need to `or` them together. Swift cannot even infer the types, so we need to write quite a bit of verbose code.

Now we can use the detector to match content in a string.

We start by defining the string we want to match in, a few regex [matching options](https://developer.apple.com/documentation/foundation/nsregularexpression/matchingoptions), and the range of the string to check in.

```
let string = "Make sure you check out my website andyibanez.com tomorrow at 8 and on Friday"
let range = NSRange(0..<string.count)

let foundContent = detector.matches(
  in: string,
  options: [], range: range)
```

Once again, this being an old API, we need to use the good ol' `NSRange` instead of the Swift's range. Other than that, nothing too bothersome here.

That will return an array of [`NSTextCheckingResult`](https://developer.apple.com/documentation/foundation/nstextcheckingresult)s, and you can just iterate over the elements:

```
foundContent.forEach { content in
  switch content.resultType {
  case NSTextCheckingResult.CheckingType.link:
    print("URL: \(content.url!)")
  case NSTextCheckingResult.CheckingType.date:
    print("DATE: \(content.date!)")
  default:
    break
  }
}
```

There are different result types for different objects. The good news is, that once you know we have a certain type in the `switch`, we can just force-unwrap the expected element. The code above prints the following in my machine:\

```
URL: http://andyibanez.com
DATE: Optional(2020-01-30 12:00:00 +0000)
DATE: Optional(2020-01-31 16:00:00 +0000)
```

Dates are in UTC, so you may need to operate on them when you  receive them. If your string contains text that can be used to to deduce the timezone, you can access the `timezone` property of `NSTextCheckingResult`.

```swift
let string = "Visit me tomorrow at 8 UTC+4"
//...
case NSTextCheckingResult.CheckingType.date:
    print("DATE: \(content.date!) \(content.timeZone)")
//...
// DATE: 2020-01-30 04:00:00 +0000 Optional(GMT+0400 (fixed))
```

Note that the link was detected in a very smart way and you didn't even need to specify the protocol.

# Conclusion

`NSDataDetector` is an old but still very powerful API. What we saw today is just the type of the iceberg of its features. This object can match other types of content as well. If you are interested, I encourage you to check out the documentation. It has some limitations, but it will for many cases.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.