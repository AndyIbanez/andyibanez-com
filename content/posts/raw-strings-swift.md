---
title: "Raw Strings in Swift"
date: 2021-04-21T07:00:00-04:00
publishDate: 2021-04-21T07:00:00-04:00
originalDate: 2021-04-19T22:17:36-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - apple
 - programming
categories:
 - development
keywords:
 - swift
 - apple
 - programming
description: "Learn what Raw Strings are in Swift and how to use them."
---

We have all worked with strings before. Printing a piece of text, or displaying some information to users in a label, can all be done in strings. But regardless of how popular strings are, they actually have a lot of complex or unknown functionality that can help developers, but they struggle to see the light of day.

In  this article, we will explore a very interesting aspect of strings in Swift: Raw Strings, what they are, and how they can be helpful to your every day job.

# Introducing Raw Strings

Raw strings (denoted by starting and ending pound symbols (`#`) before and after the quotation marks), allow us to create strings that will print exactly what you see. As you know, there are certain escape sequences that can make our strings print differently. For example, writing a `\n` will cause your string to break your string in separate lines. If you want to actually print or use a `\` character, you need to write it twice. All this functionality of strings is there to help us create memorable text. But sometimes, this behavior if not wanted.

Consider the following example:

```swift
let message = "Hey there cowboy! \n In reverse land, the current date is 2020\01\10!"
```

The first obvious problem is that your program won't compile at all, because `\1` and `\0` are not valid escape sequences. If you want to get this to compile, you have to add an extra backslash to each existing one:

```swift
let message = "Hey there cowboy! \n In reverse land, the current date is 2020\\01\\10!"
```

This now compiles, and shall you print it you will get:

```
Hey there cowboy! 
 In reverse land, the current date is 2020\01\10!
```

This is all cool and dandy, but that extra backslash was just *annoying*. If we make this a raw string, it will prevent Swift from executing escape sequences:

```swift
let message = #"Hey there cowboy! \n In reverse land, the current date is 2020\01\10!"#
```

We now have a valid string whose escape sequences have not been executed. If you print the string you will get the following output:

```
Hey there cowboy! \n In reverse land, the current date is 2020\01\10!
```

We no longer need to escape our backslashes twice, but we have lost the line break, which you may have wanted. We have not lost completely the ability to use escape sequences. Instead, every backslash you use now, that you want to be executed as a escaping sequence, must be followed by a `#` and then the sequence you want to execute.

```swift
let message = #"Hey there cowboy! \#n In reverse land, the current date is 2020\01\10!"#

print(message)
```

```
Hey there cowboy! 
 In reverse land, the current date is 2020\01\10!
```

This also means that if you intend to use string interpolation, you will need to follow the backslash with a pound.

```swift
let age = 29
let string2 = #"This year I turn \(age) years old"#
print(string)

//...

let string2 = #"This year I turn \#(age) years old"#
print(string2)
```

```
This year I turn \(age) years old // string
This year I turn 29 years old // string2
```

Finally, you can also use raw strings with multiline strings.

```swift
let badPoem =
#"""
This year \
I shall turn \
\#(age) years old
"""#

print(badPoem)
```

```
This year \
I shall turn \
29 years old
```

Keep in mind that everything you can do with raw strings, can be done without, so this is not a mandatory tool in your toolbox, but it is nice to have, especially when working with Regex.

# Conclusion

Raw strings are useful and good to have in your toolbox. While most of its uses are not immediately visible, you will *know* when you can use them, so it's good to keep them in mind.

