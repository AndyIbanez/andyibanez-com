---
title: "Swift's print in Depth"
date: 2021-04-28T07:00:00-04:00
originalDate: 2021-04-25T23:56:30-04:00
publishDate: 2021-04-28T07:00:00-04:00
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
---

Ah, `print`. Probably the most known, the most used, the most popular debugging tool, and probably the most loved line of code of all time. You have undoubtedly used print before, if not in Swift, in other languages. The vast majority of programmers have started their software building skills with a `print` or equivalent somewhere.

We have all used `print` before, but this short article is about using the function to the max - it actually has a secret or two you might not know about.

Note this article is about `print` - There are actually other useful functions for printing text in Swift (such as `debugPrint`), but this article is not about them.

# Custom Separators and Terminators

The first small thing you can do with `print` is to change the default separators and terminators.

## Separator

In case you haven't noticed, `print` takes a variable number of parameters as its first argument. You can use it to print multiple objects at once. For example:

```swift
print(1, 2, 3, 4)
// prints
// 1 2 3 4
```

By default, the separator is an empty space, but you can customize it to be anything else.

```swift
print(1, 2, 3, 4, separator: ",")
// prints
// 1,2,3,4
```

Changing the separator can be useful, as we will see later on.

## Terminators

By default, when `print` is done printing your text, it will append a newline character to each line, so further `print` sequences start on their own line.

```swift
print(1, 2, 3, 4)
print(5, 6, 7, 8)
```

This will print:

```
1 2 3 4
5 6 7 8
```

We can provide any terminator we want:

```swift
print(1, 2, 3, 4, terminator: "|")
print(5, 6, 7, 8, terminator: "|")
```

```
1 2 3 4|5 6 7 8|
```

And of course, you can use a combination of both `separator` and `terminator`.

```swift
print(1, 2, 3, 4, separator: ",", terminator: "|")
print(5, 6, 7, 8, separator: ",", terminator: "|")
```

```
1,2,3,4|5,6,7,8|
```

# Redirecting Output with the output: parameter

The last thing I want to tell you about is probably my favorite `print` feature. By default, `print` sends all its output to the standard output, which is going to be Xcode's console most of the time. But that doesn't stop us from changing this and printing somewhere else. The last optional parameter we can pass in to `print`, is `to output:`, which is an `inout` parameter of an object that conforms to `TextOutputStream`.

`TextOutputStream` has a single requirement you need to implement:

```swift
    mutating func write(_ string: String) {

    }
```

As you can see, we need to implement a `mutating func` that will give us a string.

This is interesting, because there is actually a lot we can do here. you can continue sending text to the standard output after mutating it here (by simply calling print again), or you can redirect the output somewhere entirely different.

To show you content mutation, I will reuse the `EmojiFormatter` I wrote for my [Writing Custom NSFormatters in Swift](/posts/writing-custom-nsformatters-swift/) article. This formatter will find ASCII emoticons and convert them into emojis. For example, `:-)` will get converted into `🙂`.

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

We will now create a simple `TextOutputStream` that will mutate our strings, replace some emojis and print them into the standard output:

```swift
struct EmojiLogger: TextOutputStream {
    let formatter = EmojiFormatter()
    
    mutating func write(_ string: String) {
        print(formatter.string(for: string)!)
    }
}

var logger = EmojiLogger()

print("Hi, I'm happy to meet you :-)", to: &logger)
```

This will print:

```
Hi, I'm happy to meet you 🙂
```

Of course, the true power of this is lies in the fact that we can make it do something entirely different. We could, for example, create a simple logging functionality where everything we print will be recorded on a file on disk.

To show you how this is done, I took a [Logger class](https://stackoverflow.com/a/58697135/648767) from StackOverflow (always take precautions when using third party code in your projects), and we are now going to integrate it into a `FileLogger`.

```swift
class Logger {

    static var logFile: URL? {
        guard let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else { return nil }
        let formatter = DateFormatter()
        formatter.dateFormat = "dd-MM-yyyy"
        let dateString = formatter.string(from: Date())
        let fileName = "\(dateString).log"
        return documentsDirectory.appendingPathComponent(fileName)
    }

    static func log(_ message: String) {
        guard let logFile = logFile else {
            return
        }

        let formatter = DateFormatter()
        formatter.dateFormat = "HH:mm:ss"
        let timestamp = formatter.string(from: Date())
        guard let data = (timestamp + ": " + message + "\n").data(using: String.Encoding.utf8) else { return }

        if FileManager.default.fileExists(atPath: logFile.path) {
            if let fileHandle = try? FileHandle(forWritingTo: logFile) {
                fileHandle.seekToEndOfFile()
                fileHandle.write(data)
                fileHandle.closeFile()
            }
        } else {
            try? data.write(to: logFile, options: .atomicWrite)
        }
    }
}
```

The `FileHandler` implementation now looks like this:

```swift
struct FileLogger: TextOutputStream {
    func write(_ string: String) {
        Logger.log(string)
    }
}

var logger = FileLogger()

print("This is getting written to a file.", to: &logger)
```

The first thing you will notice is that once we run this, it will not print anything to the console. This is because, this time, we are *redirecting* the output of text to a completely different place. This is really neat, as it allows you to create custom print functions that you can choose where to send their output to, whether you want to keep default console printing on, and more.

# Conclusion

`print` can be more powerful than we give it credit here. It having a native way to let us customize or completely redirect output is nothing short of amazing. Using `TextOutputStream`, we can customize many aspects about our `print` calls.

