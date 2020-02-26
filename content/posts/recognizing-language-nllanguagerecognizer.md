---
title: "Recognizing the language in a Natural Language Text with NLanguageRecognizer"
date: 2020-02-19T07:00:00-04:00
draft: false
originalDate: 2020-02-13T07:26:20-04:00
publishDate: 2020-02-19T07:00:00-04:00
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
description: "Identify the language of a body of natural language text with NLLanguageRecognizer."
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

Continuing my trend of writing about language processing, today I want to discuss about identifying the language of a body of text. This is an interesting task we can do thanks, once again, to Apple's investment in APIs linked to machine learning.

Today we will explore the [`NLLanguageRecognizer`](https://developer.apple.com/documentation/naturallanguage/nllanguagerecognizer) object. Introduced in iOS 12, this class can do a lot of language recognizing, from detecting the "dominant language" of a string, to all the possible languages.

# Introducing NLLanguageRecognizer

<hr>
**Important Note!**

Don't try to use one instance of this object through multiple threads.
<hr>

This class is actually very easy to use. It has very few methods, and the easiest task to perform is one static method away. You need to `import NaturalLanguage` to use it.

## Quickly Recognizing a Language in a String.

If all you need to do is to quickly recognize a language in a string, you can use the static `dominantLanguage(for:)` method. This method takes the string to recognize and returns an optional [`NLLanguage`](https://developer.apple.com/documentation/naturallanguage/nllanguage) object, which contains the language itself. If the string cannot recognize a language at all, it will be nil.

```swift
var stringToRecognize = "This is an awesome string."

if let lang = NLLanguageRecognizer.dominantLanguage(for: stringToRecognize) {
  print(lang.rawValue) // prints "en"
}
```

And the fun part is, because the method can return the *dominant* language, you can mix multiple languages together and it will return the one with most presence.

```swift
var stringToRecognize = "This is an awesome string. Cuando yo estaba por ahí en las calles decidí preguntar el significado de la vida"

if let lang = NLLanguageRecognizer.dominantLanguage(for: stringToRecognize) {
  print(lang.rawValue) // prints "es"
}
```

The above example uses a string with both English and Spanish. Because Spanish is the dominant language, it prints "es".

## Advanced Usage

That's probably a bad title, because using the other features o this class is not complicated at all.

First, we can detect all the languages in a string. Although this will not be accurate all the time, an instance of this class offers the `languageHypotheses(withMaximum)` method, which tries to return all the languages found in a string. The return type is a dictionary of type `[NLLanguage: Double]`. The double is the probability of each language. The `withMaximum` parameter is the maximum number of languages to return.

To use an instance instead of the static methods of this class, you have to call the `processString` method, which takes a string and returns nothing. After you call this method, `NSLanguageRecognizer`s will have its `dominantLanguage` property filled. You will also be able to use ``languageHypotheses(withMaximum)`` method. Calling `processString` is essential to do anything interesting with this class. And like you are able to tell, everything happens in the same thread, so remember not to use one instance concurrently.

The following example gets all the possible languages in the string:

```swift
var stringToRecognize = "This is an awesome string. Cuando yo estaba por ahí en las calles decidí preguntar el significado de la vida"

let langRecognizer = NLLanguageRecognizer()
langRecognizer.processString(stringToRecognize)
for (lang, perc) in langRecognizer.languageHypotheses(withMaximum: 10) {
  print("Probability of \(lang.rawValue): \(perc)")
}
```

It will output something like the following:

```swift
Probability of de: 0.0011527120368555188
Probability of sk: 0.0013781085144728422
Probability of hu: 0.001778516685590148
Probability of it: 0.003761883592233062
Probability of pt: 0.01308358833193779
Probability of nl: 0.0018757604993879795
Probability of ro: 0.0038427249528467655
Probability of hr: 0.0007617694209329784
Probability of en: 0.09818074852228165
Probability of es: 0.8707313537597656
```

You should try to give the `withMaximum` parameter a more reasonable value if you have any idea of what the dominant languages are going to be. We can observe that English and Spanish have the bigger percentages.

You can also guide the recognizer by specifying the `languageHints` and `languageConstraints` properties. I wasn't able to find much use for `languageHints` because it takes a dictionary similar to the one returned by `languageHypotheses(withMaxium:)`, but by using `languageConstraints` you can limit the languages you want to recognize.

If we add the following code to the piece of code above

```swift
langRecognizer.languageConstraints = [.english, .spanish]
```

It will print:

```
Probability of es: 0.8986690044403076
Probability of fr: 0.0
Probability of hr: 0.0
Probability of da: 0.0
Probability of en: 0.10133091360330582
Probability of cs: 0.0
Probability of fi: 0.0
Probability of de: 0.0
Probability of hu: 0.0
```

And it would be good to assign `withMaxium` to `2` here, as we know we only want to recognize two languages.

# Conclusion

Recognizing a language in iOS is as easy as using the `NLLanguageRecognizer` API introduced in iOS 12 and calling a few lines of code. The system will do its best to determine the dominant language or all the possible languages in a string, and you can use this information for natural-language apps.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.