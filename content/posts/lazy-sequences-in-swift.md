---
title: "Lazy Sequences in Swift"
date: 2020-11-25T07:00:00-04:00
originalDate: 2020-11-22T22:45:06-04:00
publishDate: 2020-11-25T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - apple
 - sequences
categories:
 - development
description: "Learn about lazy sequences in Swift and how to use them."
keywords:
 - swift
 - apple
 - sequences
---

Lazy Sequences in Swift

If you have been writing Swift for a while, you have undoubtedly used high order functions such as `.map` and `filter`. These higher order function work on any collection, and they are very useful when we want to quickly transform objects into something else, or when you want to do other operations in sequences that would otherwise take more than one line of code if you were to make them with loops.

However, applying these higher order functions to collections directly can pose some implications regarding performance and memory usage. If you have an array of 1000 elements of an object of type `X`, when you apply `map` to it, it will operate over all 1000 elements to create your new array of objects of type `Y`. Now, 1000 elements is very tiny for a computer with the power of an iPhone, but what if you have much bigger collections?

This is where lazy sequences come into play.

```Swift
struct Character {
  let name: String
}

let characters = ["Elize", "Arietta", "Anise"]

let mappedCharacters = characters.map { Character(name: $0) } // A new collection of 3 elements
let lazyMappedCharacters = characters.lazy.map { Character(name: $0) } // This won't execute any code until you need it.

print(lazyMappedCharacters[2])
```

In this example, we want to convert an array of `[String]` into `[Character]`. `mappedCharacters` will go ahead and map all the strings into Characters and store it, doing more work and using more memory, whereas, when you use `lazyMappedCharacter`, your `.map` closure won't be called immediately. Instead, the closure will be stored and it will be called *as you need it*. When we print `lazyMappedCharacters[2]`, the lazy collection will take the original collection, search for index `2`, apply the closure to it, and return it.

# When to use lazy sequences? When to use the standard sequences?

At a first glance, after you reading this, you may be tempted to just go back to all your `.map`/`.filter` calls and change them to call `.lazy` on all of them. Don't do that.

First, know that lazy sequences don't implement any sort of cache, so if you keep calling `lazyMappedCharacters[2]` over and over again, the closure will be applied over and over again and therefore the mapped value will be calculated each time.

Second, if you use `.filter` and you want to `.count` the number of elements in the resulting sequence, be aware that filter will have to go through all the elements to give you the total count. If you need to `.count`, you should use `.filter` on a non-lazy sequence. Operating on lazy collections is slower than operating on non-lazy ones, so you may think that calling `.count` in a non-lazy or lazy sequence is the same - it's not.

On the other hand, if you consider you don't have memory to spare, or you don't need to do something with all the results of a `.filter` or `.map` immediately, you could use lazy collections. For example, if you are consuming a web service and want to convert all the JSON into user objects, you can avoid mapping the JSON that is not currently visible to the user by using `.lazy`.

If you find yourself in a situation which you genuinely don't know which one to use, I'd recommend opting to use standard sequences over lazy ones. Lazy is great, but it can be seen as premature optimization for some, and while you may have some uses for it in mind, you may find yourself in the unexpected scenarios in which you do need to do that `.count` call after all, completely voiding `.lazy`'s benefits and actually writing worse-performing code.

# Conclusion

`.lazy` sequences are an interesting tool to add to your arsenal. They have performance implications that can sometimes be better or worse. Use them wisely, and don't mindlessly apply their use everywhere unless you have a good reason to.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!