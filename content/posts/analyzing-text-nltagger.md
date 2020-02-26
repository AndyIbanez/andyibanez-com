---
title: "Analyzing Natural Language Text with NLTagger"
date: 2020-02-26T07:00:00-04:00
draft: false
originalDate: 2020-02-19T07:24:23-04:00
publishDate: 2020-02-26T07:00:00-04:00
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
description: "How to use the NLTagger class to analyze natural language text."
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

Analyzing Text with NLTagger

In the past few weeks, we have explored how we can [tokenize natural language text](https://www.andyibanez.com/posts/tokenizing-nltokenizer/) and how to [recognize the language a natural language text is written in](https://www.andyibanez.com/posts/recognizing-language-nllanguagerecognizer/). This week we will continue exploring more natural language APIs provided by the `NaturalLanguage` framework. We will learn about the [NLTagger](https://developer.apple.com/documentation/naturallanguage/nltagger) class, which allows us to to analyze natural language text to find parts of speech, lexical classes, lemma, scripts, and more. This API, introduced in iOS 12, implements machine learning to work, and just like the other `NaturalLanguage` classes, is very easy to use.

# Introducing NLTagger

NLTagger is the class of the `NaturalLanguage` framework that allows us to analyze text and find its components. We have explored classes to tokenize text and to detect the language - now it's time to actually do more interesting tasks with the provided text. 

When working with `NLTagger`, you specify the components you are interested in (part of speech, lexical class, etc) by specifying an array of [`NLTagScheme`](https://developer.apple.com/documentation/naturallanguage/nltagscheme).

## Using NLTagger

### Using NLTagger With Simple Words

Define the string you want. Then, create the `NLTagger` object passing an array of `NLTaggerScheme`s. In the following example we will define a simple word string, we will analyze all of it, and we are interested in getting its lexical class (The lexical class tells whether the word is a noun, verb, or other grammatical component).

```swift
var stringToRecognize = "visit"
let tagger = NLTagger(tagSchemes: [.lexicalClass])
```

Then, assign the string you want to analyze to the tagger's `string` property. Once you do that, you will be able to get the independent tag:

```swift
tagger.string = stringToRecognize
let tag = tagger.tag(at: stringToRecognize.startIndex, unit: .word, scheme: .lexicalClass)
```

We are getting the tag of a `.word` unit. In this case, the tag is of type `otherWord`, because there is not enough context to deduce its lexical class. If you change the word to use the string `"visiting"`, then the tag will be `verb`.

As an important aside, you shouldn't this class in a multithreaded environment.

### Using NLTagger with Longer Text

We talked about identifying single words, but in a real-world case you are more interested in analyzing longer strings of natural language text. We saw how we could iterate over all the tokens when using `NLTokenizer`, and the good news is that we can do something very similar with `NLTagger`.

After you define your string, define the range you want to analyze. You can then use `NLTagger`s `enumerateTags(in:unit:scheme)` to go through all of the words:

```swift
var stringToRecognize = "I will visit you tonight. The house is empty."
let range = stringToRecognize.startIndex ..< stringToRecognize.endIndex
let tagger = NLTagger(tagSchemes: [.lexicalClass])
tagger.string = stringToRecognize
tagger.enumerateTags(in: range, unit: .word, scheme: .lexicalClass) { (tag, range) -> Bool in
  print("Word [\(stringToRecognize[range])] : \(tag!.rawValue)")
  return true
}
```

This will print something like this:

```
Word [I] : Pronoun
Word [ ] : Whitespace
Word [will] : Verb
Word [ ] : Whitespace
Word [visit] : Verb
Word [ ] : Whitespace
Word [you] : Pronoun
Word [ ] : Whitespace
// ....
```

When it comes to the English language, the `whitespace` might just be noise and you may not care about it. So we can filter it out and ignore all the whitespace, by adding a check inside function's closure:

```swift
if tag != .whitespace {
	print("Word [\(stringToRecognize[range])] : \(tag!.rawValue)")
}
```

This will correctly ignore all the whitespace characters.

```
Word [I] : Pronoun
Word [will] : Verb
Word [visit] : Verb
Word [you] : Pronoun
Word [tonight] : Noun
Word [.] : SentenceTerminator
Word [The] : Determiner
Word [house] : Noun
Word [is] : Verb
Word [empty] : Adjective
Word [.] : SentenceTerminator
```

### Working with Constrains

Now all languages may support the same tag schemes, so it may be important to know what schemes are supported in certain scenarios. For this, `NLTagger` has a `availableTagSchemes(for:language:)` class method.

The following example shows you all the possible tag schemes for English and Japanese:

```swift
let enSchemes = NLTagger.availableTagSchemes(for: .word, language: .english)
print("English")
print(enSchemes.map ({ $0.rawValue }))

let jpSchemes = NLTagger.availableTagSchemes(for: .word, language: .japanese)
print("Japanese")
print(jpSchemes.map ({ $0.rawValue }))
```

```
English
["Language", "Script", "TokenType", "NameType", "LexicalClass", "NameTypeOrLexicalClass", "Lemma"]
Japanese
["Language", "Script", "TokenType"]
```

### Utilizing NLTagger to Perform Sentiment Analysis

One interesting feature of NLTagger is that it can be used to perform sentiment analysis. If you are not familiar with this term, sentiment analysis is the application of machine learning to learn whether a piece of text can be considered "positive" or "negative".  You can think of it whether a text speaks positively or negatively about something.

To perform sentiment analysis, you need to use the `.paragraph` unit, and you must also use the `.sentimentScore` tag scheme. Other than that, its use is pretty straightforward:

```swift
var stringToRecognize = "Andy Ibanez is awesome"
let tagger = NLTagger(tagSchemes: [.sentimentScore])
tagger.string = stringToRecognize
let (sentimentScoreTag, _) = tagger.tag(at: stringToRecognize.startIndex, unit: .paragraph, scheme: .sentimentScore)
let sentimentScore = sentimentScoreTag?.rawValue ?? "0"
print("Score: \(sentimentScore)")
```

```
0.6
```

The sentiment analysis has a score from -1.0 for negative to 1.0 for positive.

Sentiment Analysis is available on iOS 13 and later.

# Conclusion

NLTagger allows you to analyze a piece of natural language text to get its components and understand how a sentence is formed. The analysis can be done in different ways for different languages. We can also perform sentiment analysis on strings starting on iOS 13. NLTagger has many features and it's easy to use.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.