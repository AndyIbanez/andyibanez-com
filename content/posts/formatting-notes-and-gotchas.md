---
title: "Formatting Notes and Gotchas"
date: 2020-09-30T07:00:00-04:00
originalDate: 2020-09-28T09:57:37-04:00
publishDate: 2020-09-30T07:00:00-04:00
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
 - iOS14
 - nsformatter
 - wwdc2020
categories:
 - development
description: "Usage notes and things to watch out for when formatting content with NSFormatters."
keywords:
 - swift
 - ios
 - tvos
 - ipados
 - watchos
 - iOS14
 - nsformatter
 - wwdc2020
---

A year ago, we talked about [using NSFormatter for formatting data in a human readable format](https://www.andyibanez.com/posts/nsformatter/). WWDC2020 brings some updates and changes to the `NSFormatter` APIs that we need to be aware of. This article will complement the NSFormatter article from last year with best practices and things to look out for.

# Improvements for Combinations of Languages and Regions.

`NSFormatter` always does its best to format the data according to the user's language and region where relevant. Apple is improving the combinations for this because it's highly common for people to set their phones in a language that is not commonly used in a given region. This is pretty exciting for me, because I live in Bolivia where people speak Spanish, but I have used my devices in English for as long as I can remember.

This does mean that you should watch out for something that *may* look unintended, but it is what makes sense in different user devices.

In general, you should never try to force the formatting of content in a way that is not set in the user's device. Ever since my previous article was published, people have been asking me about formatting data in a very specific way according to the developer's preference. Don't do this - Just keep in mind the most mundane differences (some countries separate decimals with commas, others with periods) in mind, and let the APIs do the formatting. Attempting to force an specific formatting can cause weird bugs where you least expect them.

As an example, I work for a bank, and I format currency amounts in different currencies on iOS all the time. There was very old legacy code that expected a number formatting to always have commas and replace them with periods (we use periods for separating thousands here). This code broke in many different ways if users were using different locale settings. Even when the code tried to force a specific formatting, subsequent versions of iOS broke the code.

Moral of the story: **Let NSFormatter do its jobs, and don't rely on its output to be used as input in other methods**.

In iOS 14, due to the internal improvements and further combinations of languages and regions, bugs like this are very likely to pop up often, so watch out for that.

# Templates

Actually introduced in iOS 8 (!), templates for date formatters allow us to do custom formatting and have it displayed properly for each locale.

If you have used `DateFormatter` before, you have likely used the `dateFormat` property. If you need to format in a specific locale, you should set the format with `setLocalizedDateFormatFromTemplate(_:)` instead. Setting the format this way will let the formatter set the order of the fields and any given punctuation for the final output.

Let's take controversial format: In the USA, dates are represented as month/day/year. Most countries in the world will use the more correct (I avoid saying things like this in my articles, but I will die on this hill) day/month/year.

Using `dateFormat`, you can format a date like this:

```swift
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "MM/dd/yyyy"
dateFormatter.string(from: Date()) // 09/27/2020
```

You are forcing the formatter to output a USA date. What if you wanted to format it for different locales? We fellas in South America don't like that format. For us, it should really say `27/09/2020`. If you wanted your format to look right in all possible locales, you can probably implement an if-hell and account for different locales that way.

But by setting the format with `setLocalizedDateFormatFromTemplate(_:)` instead, the formatter will do the right thing based on the locale you give it.

```swift
let dateFormatter = DateFormatter()
dateFormatter.locale = Locale(identifier: "en_US")
dateFormatter.setLocalizedDateFormatFromTemplate("MMddyyyy")
dateFormatter.string(from: Date()) // US: 09/27/2020

// ...

dateFormatter.locale = Locale(identifier: "es_BO")
dateFormatter.setLocalizedDateFormatFromTemplate("MMddyyyy")
dateFormatter.string(from: Date()) // Bolivia: 27/09/2020
```

It is very use to have dates formatted for the right locale. Grabbing the current Locale is also very easy, so there is no reason to not let your app format correctly for all possible locales.

Note that `setLocalizedDateFormatFromTemplate(_)` will ignore anything that is not a valid date specifier. This is fully expected, as it will take care of any formatting for you. This also means that the order of the fields does not matter either.

With this, there's very little reason to use the `dateFormat` property, as using it directly will produce the wrong output in many cases, especially if the developer is not aware of locale differences.

# Formatting Names

It's not unheard of for countries to treat names differently. For this reason we have `PersonNameComponentsFormatter`, which allows us to format names for each locale.

```swift
let formatter = PersonNameComponentsFormatter()
var nameComponents = PersonNameComponents()
nameComponents.familyName = "木之本"
nameComponents.givenName = "桜"
nameComponents.nickname = "Sakura"

formatter.string(from: nameComponents) // 木之本桜

formatter.style = .short
formatter.string(from: nameComponents) // Sakura

formatter.style = .abbreviated
formatter.string(from: nameComponents) // Sakura
```

This a full Japanese name. If we write it down in romaji, we can expect different output:

```swift
nameComponents.familyName = "Kinomoto"
nameComponents.givenName = "Sakura"
nameComponents.nickname = "Sakura-chan"

formatter.string(from: nameComponents) // Sakura Kinomoto

formatter.style = .short
formatter.string(from: nameComponents) // Sakura-chan

formatter.style = .abbreviated
formatter.string(from: nameComponents) // SK
```

There is interesting behavior here that you may have not noticed if you don't know the Japanese kanji here.

"木之本" is "Kinomoto", and "桜" is Sakura. The moment we wrote the name with our alphabet, the formatter output the given name first and the family name second. Using the formatter in Japanese outputs the family name first, and then the given name. Japanese people are used to putting the family name first, and the formatter has taken care of those details for us automatically.

# Formatting Lists

We can also format lists. Suppose you want to say "Apples, eggs, and pears" when you have the following array:

```swift
let items = ["apples", "eggs", "pears"]
```

You can probably think of a clever way to concatenate the first two items only with a comma and then follow the last item with the word "and". In fact doing so may not be complicated, but we have `ListFormatter` which allows us to do just that in just one line of code:

```swift
let items = ["apples", "eggs", "pears"]
ListFormatter.localizedString(byJoining: items) // apples, eggs, and pears
```

If you remove one of the elements, the formatter will do the right thing:

```swift
let items = ["apples", "pears"]
ListFormatter.localizedString(byJoining: items) // apples and pears
```

In Spanish, we can use either "y" or "e" when we want to say the word "and". When we are listing elements like this, we will use "y" the vast majority of the time, but when the sound of the letter following our "and" has an "e" ("i") sound, we use "e" (read as "eh") instead.

In the following example, we are talking about two civilizations that lived way before us: The Aymaras and Incas. If your phone was set to Spanish and you were to format this array, depending on the order of the words, you would get a different output for "and":

```swift
let historic = ["aymaras", "incas"]
ListFormatter.localizedString(byJoining: historic) // aymaras e incas

//...

let historic = ["incas", "aymaras"]
ListFormatter.localizedString(byJoining: historic) // incas y aymaras
```

Starting on iOS 14 and the rest of the OSes introduced in WWDC2020, `ListFormatter` will use the right grammatical rules for your lists. In earlier versions you may get unexpected results for some lists.

# Numbers

Like we mentioned earlier, different locales separate decimals and thousands with a different symbol. Most English people will use commas for thousands, whereas in Spanish speaking countries we normally separate with periods.

This can be done correctly with `NumberFormatter`.

```swift
let numFormatter = NumberFormatter()
numFormatter.numberStyle = .decimal
numFormatter.string(from: 32.823) // 32.823 (English), 32,823 (Spanish)
```

# Conclusion

Formatting text sounds like an easy thing, until you consider different languages and locales do things differently. As a bilingual developer, I have always been aware that languages have differences with how they should display numbers and other kinds of data, and I always thought it was a challenge to adopt to those rules. That said, Apple is making it easier to format different content and their formatters are improving to support more especial case scenarios and particular grammar rules.

This article is based on WWDC2020's [Formatters: Made data human-friendly](https://developer.apple.com/videos/play/wwdc2020/10160/) session.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.
