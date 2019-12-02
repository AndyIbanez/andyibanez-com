---
title: "Formatting Content with NSFormatter"
date: 2019-09-25T07:00:00-04:00
originalDate: 2019-09-21T17:21:42-04:00
draft: false
publishDate: 2019-09-25T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - macos
 - tvos
 - watchos
 - nsformatter
categories:
 - development
description: "Learn to use the different NSFormatter subclasses to format data in a human-readable way."
keywords:
 - swift
 - nsformatter
 - ios
 - tvos
 - ipados
 - watchos
---

Very often, we need to deal with data in a "raw" format that, if displayed directly to the user, it makes little sense to them. This kind of data includes a date timestamp, the number of bytes in a big file, or numbers with no rounding a bunch of decimals. There is a lot of data like this, and we need to be able to format it and show it to the user.

In all my years as a programmer, I have seen a lot of "hacky ways" to retrieve and parse content such as dates and file sizes. Sometimes, they were good, but more often than not they were extremely verbose code and unnecessary. NSFormatter has a lot of advantages, including providing localized representations where relevant.

# Introducing NSFormatter

NSFormatter is considered an abstract class. You should never use it directly, but you can subclass it or use the specializations already provided by Foundation. This class helps create textual representation of values, and it can else help you validate and interpret such values.

In this article, we will explore a few existing `NSFormatter`s and when you may want to use them.

# Existing Formatters

There's a bunch of formatters that are already provided by Foundation. These include:

* [ByteCountFormatter](https://developer.apple.com/documentation/foundation/bytecountformatter): To represent digital filesizes of data.
* [DateFormatter](https://developer.apple.com/documentation/foundation/dateformatter): For formatting dates.
* [DateComponentsFormatter](https://developer.apple.com/documentation/foundation/datecomponentsformatter): For formatting individual components, like "2 days". You can also use this to create strings such as "2 days ago" or "3 days remain".
* [DateIntervalFormatter](https://developer.apple.com/documentation/foundation/dateintervalformatter): For formatting time ranges.
* [EnergyFormatter](https://developer.apple.com/documentation/foundation/energyformatter): You can use this to format energy values, from joules and kilojoules, to calories and kilocalories.
* [LengthFormatter](https://developer.apple.com/documentation/foundation/lengthformatter): Provides localized descriptions for distance and height units. 
* [MassFormatter](https://developer.apple.com/documentation/foundation/massformatter): Provides formatting for mass and weight values.
* [NumberFormatter](https://developer.apple.com/documentation/foundation/numberformatter): For formatting numbers and their textual values.
* [PersonNameComponentsFormatter](https://developer.apple.com/documentation/foundation/personnamecomponentsformatter): Provides localized descriptions for the components in a person's name.
* [ISO8601DateFormatter](https://developer.apple.com/documentation/foundation/iso8601dateformatter): Introduced in iOS10, this handy formatter allows you to quickly work with dates returned in ISO8601 format. Handy for those web services that don't return timestamps.

These are just some of the formatters that Cocoa already provides for you, but there's more. In this article we will not explore all of them. We will see the ones that you are very likely to need at some point.

## ByteCountFormatter

This formatter allows you to represent file sizes in bytes in strings with their respective units. In other, it allows you to display `1_000_000_000` bytes as `1GB`.

The basic usage looks like this:

```swift
let byteFormatter = ByteCountFormatter()
byteFormatter.countStyle = .decimal
byteFormatter.includesUnit = true
byteFormatter.allowedUnits = [.useGB]
let text = byteFormatter.string(fromByteCount: 1_000_000_000)
```
A bit of discussion is in order.

First, as you may know, one gigabyte is not equivalent to 1,000,000,000 bytes. It's actually 1,073,741,824 bytes. As a convenience, we tend to just round to the nearest number when we talk about computer data file sizes, and even storage media manufactures of media do the same.

If you need the formatter to be precise and not consider `1,000,000,000` bytes as 1 gigabyte, you can change the `countStyle` property to use `.binary` instead of `.decimal`.

```swift
byteFormatter.countStyle = .binary
//...
let text = byteFormatter.string(fromByteCount: 1_000_000_000) // This will now print 0.93GB
```

Now the formatter is using the right byte count with no estimations to do all its formatting. If you format the value `1_073_741_824` with the new configuration, you will get `1GB`, which is expected.

```swift
byteFormatter.countStyle = .binary
//...
let text = byteFormatter.string(fromByteCount: 1_073_741_824) // 1GB
```

Going back to the previous example, the one where the formatter prints `0.93GB`, you can configure this. It may not make sense to show a value like this for content that is less than one gigabyte, so how can we fix it? Using the `allowedUnits` property, you can specify all the units the formatter should use for formatting. Because we only have `.useGB` on it right now, it will only use gigabytes, no matter how big or small the value is.

You can specify as many units as you want here, and the formatter will know what unit to format the data with.

```swift
byteFormatter.countStyle = .binary
byteFormatter.includesUnit = true
byteFormatter.allowedUnits = [.useGB, .useMB]
let text = byteFormatter.string(fromByteCount: 1_000_000_000) // 953.7MB
```

We are now telling the formatter to use GB and MB, and it will automatically format it depending on the size. Because `1_000_000_000` bytes make up less of a gigabyte when using the `.binary` count style, the formatter knows to format it as megabytes instead of gigabytes.

This formatter is really powerful and very easy to use. Feel free to play around with its properties. You can prevent it from showing the unit by setting `includesUnits` to `false`. You can choose to show the bytes value along with the formatted value with the `includesActualByteCount` property. In general, there's a lot you can do, but covering everything will create a very long article!

## DateFormatter

You will very often want to format dates to be displayed to the user. DateFormatter gives us the power to format any date and output it in absolutely any format we want.

The basic usage looks like this:

```swift
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "EEEE, MMM d, yyyy"
dateFormatter.timeZone = TimeZone(identifier: "America/La_Paz")!
let formattedDate = dateFormatter.string(from: now) // Saturday, Sep 21, 2019
```

The `dateFormat` property takes a string in the format you want. Memorizing this format is insane, so luckily there's resources out there that list each placeholder and how to use them, such as [nsdateformatter.com](https://www.nsdateformatter.com).

You can specify the `timeZone` the dates will be formatted for. 

Once again, this is a very powerful and easy to use formatter, but it has a lot of options, even more than `ByteCountFormatter`. Explore the other properties so that you can build the formatter that you need. You will probably end up using the `locale` property often.

## DateComponentFormatter

This formatter formats quantities of time. You can use it to display how much time has passed since a given date. You can use it to format the amount of time between two `Date` objects.

Basic usage looks like this:

```swift
let dateComponentsFormatter = DateComponentsFormatter()
dateComponentsFormatter.allowedUnits = [.hour, .minute]
dateComponentsFormatter.unitsStyle = .brief
dateComponentsFormatter.string(from: oldDate, to: now) // 3hr 29min
```

This is yet again another flexible and powerful formatter. You can configure the allowed units, the style, and much more. Many modern social networks show you the relative time of new content until a certain amount of time has passed, and this formatter allows you to do the same.

# A Word On Performance

Creating Formatters is a very expensive operation, and you may want to use them in situations when you will undoubtedly create many of them. If you are writing a calendar application, you may want to use `DateFormatter` to neatly display the time of events in a `UITableView`. The problem with this is that your first attempt at implementing this will likely create a new formatter every time a new cell is queued.

If possible, create a singleton or store your existing formatters somewhere.

In an app I have worked on, I had various different date formatters for various different contexts, so I had to create many of them. I also had to display them in table views. To get as much performance as possible, I created a singleton called `DateFormatters` that stores a `[String: DateFormatter]` dictionary, and it exposes the formatters through a subscript. The subscript takes the format you want to format a date in, and it returns a formatter for it. If a format for that format already exists, it returns it, otherwise, it creates a new one. It looked like this:

```swift
class DateFormatters {
    var formatters = [String: DateFormatter]()
    
    static let shared = DateFormatters()
    
    private init() {}
    
    public subscript(dateFormat: String) -> DateFormatter {
        if let formatter = formatters[dateFormat] {
            return formatter
        }
        
        let newFormatter = DateFormatter()
        newFormatter.timeZone = TimeZone(identifier: "America/La_Paz")!
        newFormatter.dateFormat = dateFormat
        formatters[dateFormat] = newFormatter
        return newFormatter
    }
}
```

The app was for my country only, so I could use the same `TimeZone` for all formatters, and whenever I needed a new formatter, I used it like this:

```swift
let dateAndHour = DateFormatters.shared["EEEE, MMM d, yyyy HH:mm:ss"]
let dateAndHourString = dateAndHour.string(from: now) // Saturday, Sep 21, 2019 17:11:09

let justDate = DateFormatters.shared["EEEE, MMM d, yyyy"]
let justDateString = justDate.string(from: now) // Saturday, Sep 21, 2019

let justHour = DateFormatters.shared["HH:mm:ss"]
let justHourString = justHour.string(from: now) // 17:11:09

let justHourComponent = DateFormatters.shared["HH"]
let justHourComponentString = justHourComponent.string(from: now) // 17
```

If at some point I needed a formatter with a format it had already used, it would retrieve it from the dictionary rather than creating a new one.

# Conclusion

`NSFormatter` provides many powerful subclasses for formatting data into human-readable form. There's many of them provided by the framework, and you can create your own subclasses if you need to.

They have performance implications though, so you should use them carefully, specially if used in table views or other reusable components.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.