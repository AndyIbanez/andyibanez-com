---
title: "Finding Related Words with NLEmbedding"
date: 2020-03-04T07:00:00-04:00
draft: false
originalDate: 2020-02-28T15:56:48-04:00
publishDate: 2020-03-04T07:00:00-04:00
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
description: "Learn about the NLEmbedding class in iOS."
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

There may be cases in which you need to find related words to others. With the [`NSLEmbedding`](https://developer.apple.com/documentation/naturallanguage/nlembedding) class, you can find related strings based on the proximity of their vectors.

# Using NLEmbedding

Using `NLEmbedding` is very straight forward. A simple task is to get an array of related words, which come as an array of `(String, NLDistance)` back.

The distance between words tells you how "related" they are 

```swift
let embedding = NLEmbedding.wordEmbedding(for: .english)
let foundWords = embedding!.neighbors(for: "family", maximumCount: 3)
print(foundWords)
```

In this example, it will print:

```
[("life", 0.8834981918334961), ("child", 0.8971378207206726), ("parent", 0.8989249467849731)]
```

The shorter the number, the shorter the distance is between your word and the related words. Said another way, if the distance is too small, the smaller the distance, the more related the words are.

What I find interesting is how expansive its vocabulary is. If you pass in the word `"anime"`, you will get:

```swift
[("manga", 0.7037466764450073), ("comic", 0.9883537888526917), ("fanfic", 1.0021240711212158)]
```

There's a few more things you can do with this class. You can, for example, find the distance between two words to see how related they are:

```swift
print(embedding!.distance(between: "house", and: "potato"))
```

```text
1.2890079021453857
```

There is a version of this method that takes a [`NLDistanceType`](https://developer.apple.com/documentation/naturallanguage/nldistancetype), but it doesn't seem to be very useful right now, as this enum only has one value at this time.

What we saw here are some common tasks that can be done with this class. They are likely to cover most of your needs, but feel free to explore the documentation to find more uses for it.

# Conclusion

NLEmbedding can be used to find "related" words to each other. You know how google can show alternatives for words related to the ones used in your query? That's the magic of natural language embedding. Use this class if you need to support similar functionality in your apps.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.