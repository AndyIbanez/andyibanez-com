---
title: "Writing Custom NSFormatters in Swift"
date: 2020-10-14T07:00:00-04:00
originalDate: 2020-10-12T10:40:08-04:00
publishDate: 2020-10-14T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - nsformatter
 - swift
 - programming
 - apple
 - ios
 - ipados
categories:
 - development
description: "Learn how to write your own NSFormatter subclasses in Swift."
keywords:
 - nsformatter
 - swift
 - programming
 - apple
 - ios
 - ipados
---

Last year we explored [some NSFormatters and how to use them](https://www.andyibanez.com/posts/nsformatter/). We also explored some formatters [introduced in iOS 13](https://www.andyibanez.com/posts/formatting-relative-dates-relativedatetimeformatter/). Finally, a few weeks ago [we learned about yet more formatters, and how to better use the ones we already had](https://www.andyibanez.com/posts/formatting-notes-and-gotchas/). In short, we have explored how powerful NSFormatter is. One thing we haven't done yet though, is to write our own custom `NSFormatter` subclass.

# NSFormatter

`NSFormatter` is an abstract class. All formatter classes inherit from it. In Swift, everything we need about it is `open`, so we can create our own `NSFormatters` with ease.

## Overriding NSFormatters

The class comes with many methods you can override, but you must, at the very least, override the following:

* `func string(for obj: Any?) -> String`
* `getObjectValue(_ obj: AutoreleasingUnsafeMutablePointer<AnyObject?>?, for string: String, errorDescription error: AutoreleasingUnsafeMutablePointer<NSString?>?) -> Bool`

These look a bit messy, especially the second method. The second method will return us a formatted object. It can be a string, or anything else that makes sense in the context of your app.

Other methods you can override include:

* `attributedStringForObjectValue:withDefaultAttributes`: Suppose you have a string that is supposed to represent a big title and a smaller title underneath. You can use this formatter to format such strings.
* `editingStringForObjectValue:`: You can override this when you are editing a string and the string your users see are different. By default, this will call `stringForObjectValue:`.

There's a few other ones, but they are either only useful in macOS or very complicated to implement (beyond this article). We won't implement all of the methods, but be aware they exist so you can write a formatter that fits your needs.

### EmojiFormatter

To show how to write our own formatters, we will create `EmojiFormatter`. The formatter will take strings with old-school emoticons - such as `:-), :-(, :-|, ;-(` - and it will replace them with an actual emoji - like `🙂, ☹️, 😐, 😢`.

Because this formatter operates on strings and returns strings, we will define two things before moving on:

* The String representation will be the string that contains the Emoji. For example, `I'm happy to talk to you 🙂`.
* The original object will be the string with raw ASCII emoticons. `I'm happy to talk to you :-)`.

#### Implementing Emoji Formatter

To write the class, start by subclassing `Formatter` and overriding the two mandatory methods:

```swift
class EmojiFormatter: Formatter {
    override func string(for obj: Any?) -> String? {

    }
    
    override func getObjectValue(_ obj: AutoreleasingUnsafeMutablePointer<AnyObject?>?, for string: String, errorDescription error: AutoreleasingUnsafeMutablePointer<NSString?>?) -> Bool {

    }
}
```

Next, we will add a property to make the formatting a bit easier through ASCII/emoji mapping.

```swift
let emojiMapping = [
    ":-)": "🙂",
    ":-|": "😐",
    ":-(": "☹️",
    ";-(": "😢"
]
```

Next, implement two methods: One will replace ASCII with emoji, and the other will replace emoji with ASCII.

```swift
func replaceAsciiWithEmoji(in string: String) -> String {
    var rawString = string
    emojiMapping.forEach {
        rawString = rawString.replacingOccurrences(of: $0, with: $1)
    }
    return rawString
}

func replaceEmojiWithAscii(in string: String) -> String {
    var rawString = string
    emojiMapping.forEach {
        rawString = rawString.replacingOccurrences(of: $1, with: $0)
    }
    return rawString
}
```

Finally, we need to implement the overriden methods in order to use this formatter. They are pretty straightforward.

```swift
override func string(for obj: Any?) -> String? {
    if let string = obj as? String {
        return replaceAsciiWithEmoji(in: string)
    }
    return nil
}

override func getObjectValue(_ obj: AutoreleasingUnsafeMutablePointer<AnyObject?>?, for string: String, errorDescription error: AutoreleasingUnsafeMutablePointer<NSString?>?) -> Bool {
    obj?.pointee = replaceEmojiWithAscii(in: string) as AnyObject
    return true
}
```

The last method looks complicated. It's important to remember that `(NS)Formatter` is an Objective-C class, and thus it has kept most of its original interfaces when exposing it to Swift. The obj `AutoreleasingUnsafeMutablePointer` contains a reference to the object that will be returned upon converting the string into something. In this case, it's a string.

##### Formatter Usage

With that, our formatter is done. We can get an ASCII representation of an emoji string like so:

```swift
let emojiFormatter = EmojiFormatter()
print(emojiFormatter.string(for: "I'm happy to talk to you 🙂 how you doin'? 😐") ?? "") // I'm happy to talk to you :-) how you doin'? :-|
```

To get a "formatted" emoji string - that is, to convert its ASCII representations to actual emoji, we need to do a little bit of more low-level stuff and bridging:

```swift
var formattedEmojiStringContainer: AnyObject?
var fstring = "I'm happy to talk to you 🙂 how you doin'? 😐"
var errorDescription: NSString?
emojiFormatter.getObjectValue(&formattedEmojiStringContainer, for: fstring, errorDescription: &errorDescription)

print(formattedEmojiStringContainer!) // I'm happy to talk to you :-) how you doin'? :-|
```

Of course, this is less than ideal, and all formatters included in Foundation contain methods so developers don't have to do that dirty work themselves. Before continuing, we are going to write a `rawString(for emojiString: String)` methods that takes a String with Emoji and turns them into ASCII emojis.

```swift
    public func rawString(for emojiString: String) -> String? {
        var formattedEmojiStringContainer: AnyObject?
        getObjectValue(&formattedEmojiStringContainer, for: emojiString, errorDescription: nil)
        return formattedEmojiStringContainer as? String
    }
```

And now we have a nice interface to formate ASCII emojis into actual emojis.

```swift
print(emojiFormatter.emojiString(for: "I'm happy to talk to you 🙂 how you doin'? 😐") ?? "") // I'm happy to talk to you :-) how you doin'? :-|
```

And that's how you create a basic formatter! In a future article, we are going to create a more powerful formatter that will show us everything we can do with `NSFormatter`.

A reference of the entire class we wrote in this article is below:

```swift
class EmojiFormatter: Formatter {
    
    // MARK: - User facing methods
    
    public func rawString(for emojiString: String) -> String? {
        var formattedEmojiStringContainer: AnyObject?
        getObjectValue(&formattedEmojiStringContainer, for: emojiString, errorDescription: nil)
        return formattedEmojiStringContainer as? String
    }

    // MARK: - Emoji Mapping
    
    let emojiMapping = [
        ":-)": "🙂",
        ":-|": "😐",
        ":-(": "☹️",
        ";-(": "😢"
    ]
    
    func replaceAsciiWithEmoji(in string: String) -> String {
        var rawString = string
        emojiMapping.forEach {
            rawString = rawString.replacingOccurrences(of: $0, with: $1)
        }
        return rawString
    }
    
    func replaceEmojiWithAscii(in string: String) -> String {
        var rawString = string
        emojiMapping.forEach {
            rawString = rawString.replacingOccurrences(of: $1, with: $0)
        }
        return rawString
    }
    
    // MARK: - Overriden methods
    
    override func string(for obj: Any?) -> String? {
        if let string = obj as? String {
            return replaceAsciiWithEmoji(in: string)
        }
        return nil
    }
    
    override func getObjectValue(_ obj: AutoreleasingUnsafeMutablePointer<AnyObject?>?, for string: String, errorDescription error: AutoreleasingUnsafeMutablePointer<NSString?>?) -> Bool {
        obj?.pointee = replaceEmojiWithAscii(in: string) as AnyObject
        return true
    }
}
```

## Why NSFormatter? Why not create custom formatting classes?

Seeing this example, you may be thinking that it may be easier to just create an `EmojiFormatter` that doesn't inherit from `(NS)Formatter`. After all, basic formatters are just going to need string replacing at most. So what's the point?

One big advantage of using `NSFormatter` is that there's multiple places all over Foundation and the rest of the iOS, macOS, and other Apple Platforms APIs. Because they take an `NSFormatter`, you can pass them any custom formatters.

Even SwiftUI has APIs that take formatters. As of WWDC2020, you can interpolate in SwiftUI strings that take a formatter.

```swift
Text("\("I'm happy to talk you :-)" as NSObject, formatter: EmojiFormatter())")
```

Other than that cast to `NSObject`, this is very straightforward to use, and you can expect multiple places all over Apple's technologies that take formatters and you can pass your own.

![EmojiFormatter in SwiftUI](/img/emoji_formatter_swiftui.png)

# Conclusion

Formatters never cease to amaze us. Writing our own can be very easy, but it has everything you need to write more complex formatters. There's various APIs that can take formatters throughout all the APIs in Apple's platforms, and thus subclassing (NS)Formatter to get that default functionality can be very rewarding.

