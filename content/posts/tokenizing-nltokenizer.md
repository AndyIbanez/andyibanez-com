---
title: "Tokenizing Natural Language into Semantic Units in iOS"
date: 2020-02-12T07:00:00-04:00
draft: false
originalDate: 2020-02-10T19:22:30-04:00
publishDate: 2020-02-12T07:00:00-04:00
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
description: "Learn how to use the NLTokenizer Class to separate natural language strings into components"
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

Working with Natural Language is possible thanks to machine learning. Starting on iOS 12, Apple has provided many APIs just for this task. In this article we will explore how to use [`NLTokenizer`](https://developer.apple.com/documentation/naturallanguage/nltokenizer) to separate natural language text into its proper units.

# Introduction to Natural Language Tokenizing

If you are not familiar with the inner workings of Natural Language processing, tokenizing simply means that we separate a string and analyze it to find its semantic units. If you are writing a program that processes text, you may be tempted to split the string using a separator. For example, if you wanted to get all the words in a natural sentence string in an array, you would write something like this:

```swift
"This should be separated into words using the space".split(separator: " ")
```

And this naive approach *may* work under very specific constraints. Specifically, you know it will work if the language you are using is English. But languages such as Japanese don't separate their sentences with spaces. So what do you do if you need to support different languages? This is where the tokenizer comes into play.

# Using NLTokenizer

Start by importing the `NaturalLanguage` framework.

```Swift
import NaturalLanguage
```

To create the `NLTokenizer` object, call the `NLTokenizer(using:)` method, to which you have to specify what `NLTokenUnit` you want to use. At the time of this writing, you can use `.document`, `.paragraph`, `.sentence`, `.word`. The unit allows you to specify the linguistic unit. If you choose `.word`, the tokenizer will take every word and give it back to you. As you use the other units, you will get bigger portions of text.

In the example below, we will tokenize a simple sentence.

```swift
var stringToTokenize = "It was many and many a year ago, in a kingdom by the sea."

let tokenizer = NLTokenizer(unit: .sentence)
tokenizer.string = stringToTokenize
```

Since we have a single sentence, we will use the `.sentence` unit, and the first phrase of my favorite poem ([Annabel Lee](https://en.wikipedia.org/wiki/Annabel_Lee), by Edgar Allan Poe).

It's worth noting that you can specify the language to use. By default the system will use your default language, but you can specify a different language. To be more explicitly, I will set the language to English.

```swift
tokenizer.setLanguage(.english)
```

And we can finally go through the words:

```swift
tokenizer.enumerateTokens(in: fullStringRange) { (range, attributes) -> Bool in
  print(stringToTokenize[range])
  return true
}
```

The return indicates the enumerator if it should continue going through each token. If you return false, the enumerator will stop.

The output for the above code is the following:

```
It
was
many
and
many
a
year
ago
in
a
kingdom
by
the
sea
```

## Tokenizing Different Languages

Some languages don't separate words in a sentence the same way, so it can be expected that they may also separate other words with different symbols. We will now tokenize a Japanese string into words.

```swift
var stringToTokenize = "私は毎日太っています"

let tokenizer = NLTokenizer(unit: .word)
tokenizer.string = stringToTokenize
tokenizer.setLanguage(.japanese)
let fullStringRange = stringToTokenize.startIndex..<stringToTokenize.endIndex

tokenizer.enumerateTokens(in: fullStringRange) { (range, attributes) -> Bool in
  print(stringToTokenize[range])
  return true
}
```

Output:

```
私
は
毎日
太っ
て
い
ます
```

So, even if Japanese is a "run-on" written language, where there's no visual way to separate words from each other, the tokenizer knows where it word begins and ends.

<hr>
**Important Note!**

If any point you have troubles with the tokenizer, and the enumeration function doesn't give you anything, try setting the language *before* the string. For some reason, at least with the `NSLanguage` being `.spanish`, I had to set the language first.
</hr>

# Conclusion

Separating a string with a token is a common task, but doing it naively will not account for all possible cases. `NLTokenizer` is very smart as it can process words in many languages and already knows about a lot of special cases. If you are tokenizing strings in natural language, you should use this class instead of trying to do it manually.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.