---
title: "Formatting Relative Dates With RelativeDateTimeormatter"
date: 2019-12-11T07:00:00-04:00
originalDate: 2019-12-09T16:12:06-04:00
publishDate: 2019-12-11T07:00:00-04:00
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
 - iOS13
 - nsformatter
 - relativedatetimeformatter
 - wwdc2019
categories:
 - development
description: "Learn how to perform background tasks in your iOS App."
keywords:
 - swift
 - ios
 - tvos
 - ipados
 - watchos
 - iOS13
 - nsformatter
 - relativedatetimeformatter
 - wwdc2019
---

Formatting Relative Dates With RelativeDateTimeFormatter

A few weeks ago we talked about [formatting content with NSFormatter](https://www.andyibanez.com/posts/nsformatter/), an abstract class from which multiple formatting classes inherit from to allow you to format different kinds of data in a human-readable form. `NSFormatter` is not only a class you can inherit from yourself, but as iOS evolves, more formatters will be added to its family. This week, we will explore a new member of this family introduced in iOS 13: `RelativeDateTimeFormatter`.

At the time of this writing, there's no documentation on this yet (this article is being written on Dec 2, 2018, so yeah...), so I will be happy to explore how to use it with you.

# RelativeDateTimeFormatter VS. Similar Formatter Classes

In the article linked above, we already used a formatter that lets you get the relative amount of time between different date times (`DateComponentFormatter`), so what's the difference with this one?

RelativeDateFormatter allows you to present your users with date time data in an even friendlier manner, being able to show strings such as "tomorrow" (instead of "1d"), "yesterday" (instead of -1d), and even "5 minutes ago" or "in 10 hours".

# Using RelativeDateTimeFormatter

## Basic Formatting

Using this new formatter is very easy. It provides three methods which you can use to do the formatting itself:

* `localizedString(fromTimeInterval:)`: This takes a `TimeInterval` and formats the difference between the current time in the user's device and the passed interval<sup>[1]</sup>.
* `localizedString(for:relativeTo:)`: You can use this method to get the time difference between two different dates.
* `localizedString(from:)`: This methods takes a `DateComponents` object, so you can easily construct objects and check their time difference relative to the current time on the device.<sup>[2]</sup>

### Simple Usage: `localizedString(fromTimeInterval:)`

In its simplest form, you can get formatted text out of this formatter with two lines of code. We will use `localizedString(fromTimeInterval)` to show you how to do this:

```swift
let relativeDtf = RelativeDateTimeFormatter()
relativeDtf.localizedString(fromTimeInterval: 60.0) // "in 1 minute"
```

This simple line will print `in 1 minute` because internally, the formatter will use the device's time, and it will compare it with the time 60 seconds from now.

You can also calculate the time in the past by passing in negative values. If you pass in `-60.0` to the function, it will print exactly what you expect:

```swift
relativeDtf.localizedString(fromTimeInterval: -60.0) // prints "1 minute ago"
```

Simple enough, right?

### Difference between two dates.


If you want to calculate the difference between two dates (that do not necessarily involve the device's current time), you can use the `localizedString(for:relativeTo:)` function.

I cannot find a case where you want to get two unrelated dates and get a relative formatting between them. For example, I can get the difference between my birthday this year (May 20, 2019) and Christmas (May 25, 2019), but the output does not make any sense, contextually:

```swift
let andysBday2019 = Date(timeIntervalSince1970: 1558310400.0)
let christmas2019 = Date(timeIntervalSince1970: 1577232000.0)

let relativeDtf = RelativeDateTimeFormatter()
relativeDtf.localizedString(for: andysBday2019, relativeTo: christmas2019) // prints "7 months ago" (Was my birthday really 7 months ago?)
```

You can probably find a use for it.

Instead, you can use this to calculate the difference between today and another date using `Date` objects. We will calculate the time between `now` and Christmas.

```swift
let relativeDtf = RelativeDateTimeFormatter()
relativeDtf.localizedString(for: christmas2019, relativeTo: now) // prints "in 3 weeks"
```

Using this formatter you can get the relative time difference very easily.

## Formatter Configuration

The example we saw so far were simple, and they will probably suit the vast majority of your needs. But you can configure the formatter to spell out the difference, to use abbreviations, and more.

To do this, simply change the formatter's `unitsStyle` property to any of the following:

* `.abbreviated`: *3 min. ago*
* `.full`: *3 minutes ago*
* `.short`: *3 min. ago* <sup>[3]</sup>
* `‌.spellOut`: *three minutes ago*

If you want to quickly see what would a difference look like, you can use this code. It will print your dates in each `unitStyle`.

```swift
let relativeDtf = RelativeDateTimeFormatter()
[
  RelativeDateTimeFormatter.UnitsStyle.abbreviated,
  .full,
  .short,
  .spellOut
  ].forEach {
    let formatted = relativeDtf.localizedString(for: threeMinutesAgo, relativeTo: now)
    relativeDtf.unitsStyle = $0
    print(formatted)
}
```

You can use other formatting options, such as `.formattingContext`, which lets you specify where in a sentence would the formatted text appear.

You can also specify a `.dateTimeStyle`. If you specify `.named`, you will get something like "tomorrow", whereas if you specify `.numeric`, you will get "in 1 day".

```swift
relativeDtf.dateTimeStyle = .named
relativeDtf.unitsStyle = .abbreviated
relativeDtf.localizedString(from: DateComponents(day: 1)) // "tomorrow"

relativeDtf.dateTimeStyle = .numeric
relativeDtf.unitsStyle = .abbreviated
relativeDtf.localizedString(from: DateComponents(day: 1)) // "in 1 day"
```

# Conclusion

Formatters are powerful, and as time progresses, we will see more and more. In this article we explored the new `RelativeDateTimeFormatter` in iOS 13 and how to use it, so you can start implementing friendly texts for your relative date-time strings.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.

###### Foot Notes
\[1\]: I actually don't know if formats the date relative to the user's device. The API is not documented, but that's what I have observed.
\[2\]: Just like the note above, it's not documented if the formatting occurs relative to the device's time, but my observations point to yes.
\[3\]: I looked and looked and couldn't find an input that produced something different. Hopefully the docs will be up soon.