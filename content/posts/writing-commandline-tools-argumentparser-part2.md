---
title: "Writing Command Line Tools in Swift Using ArgumentParser, Part 2: Validation & Errors"
date: 2020-03-25T07:00:00-04:00
draft: false
originalDate: 2020-03-18T12:18:03-04:00
publishDate: 2020-03-25T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ArgumentParser
categories:
 - development
description: "Performing validation on ArgumentParser commands."
keywords:
 - swift
 - programming
 - apple
 - ArgumentParser
---

[Last week](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part1/) we explored how we can build a simple command line tool. We learned how to use `@Argument`, `@Option`, and `@Flag` as the building blocks for `ArgumentParser` command line tools. We we saw last week was enough to build many simple tools, but there's still a lot to explore, and cool things to learn.

This week we will learn about input validation and errors, so we can build better tools that take more constrained parameters when relevant.

# ArgumentParser Validation

ArgumentParser has all the facilities you need to validate your input, both before you need them and when exception occurs when you are using it.

ArgumentParser allows you to perform two types of validation: Pre-Running Validation, which lets you check your arguments before your `run()` function is reached, and Post-Validation errors, which allows you to throw errors when an exception occurs with an otherwise valid input.

## Command-Line Input Validation

The framework already does a lot of validation for you for free. It will validate the data types you are passing so they conform to the types specified in each property wrapper.

But you may want to do some additional validations that the framework can't do. Consider our `CharacterCount` tool from the [last article](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part1/). Suppose we want to restrict it to strings that are at least 3 character long.

`ParsableCommand` has a `validate()` method that you can implement, and you can do any custom validations within it.

To use it, implement custom checks and throw `ValidationError`s when the condition fails.

```swift
struct CharacterCount: ParsableCommand {
  @Argument(help: "String to count the characters of") var string: String
  
  mutating func validate() throws {
    if string.count < 3 {
      throw ValidationError("'string' must contain at least 3 characters.")
    }
  }
  
  func run() {
    print(string.count)
  }
}
```

Of course, you could naively do the validation within `run` itself, but implementing `validate` lets you do separation of concerns way easier.

Now when you try to run this program with a string with less than 3 characters, you will get an error like this:

```
./CharacterCount "hi"
Error: 'string' must contain at least 3 characters.
Usage: character-count <string>
```

For the record, validation is not limited to `@Argument`. You can use it with `@Option` and `@Flag` as well.

## Post-Validation Errors

We can use the `validate()` method to ensure additional constraints are fulfilled before our program runs, but what happens when the conditions are fine, but something else fails?

Consider a program that takes a path to a local file. The framework will take care of validating that the user is indeed passing you a string which is the string to the file, but you can't check if the file is in a valid format until you try opening it.

To write checks against this cases, you can use a version of the `run` function that `throws` errors. 

```swift
struct CharacterCount: ParsableCommand {
  @Argument(help: "File to count the characters of") var filePath: String
  
  func run() throws {
    let contents = try String(contentsOfFile: filePath, encoding: .utf8)
    print(contents.count)
  }
}
```

Trying to run the program with an invalid file will produce the following output:

```swift
./CharacterCount path_to_file
Error: Error Domain=NSCocoaErrorDomain Code=260 "The file “path_to_file” couldn’t be opened because there is no such file." UserInfo={NSFilePath=path_to_file, NSUnderlyingError=0x7fc2ec40ebe0 {Error Domain=NSPOSIXErrorDomain Code=2 "No such file or directory"}}
```

Of course, because this will throw `Error`s to the console, there's cases when whatever it prints may be cryptic for your users, so you should minimize throwing errors directly and only use them when there's absolutely no other way to check for them before hand.

In our specific program, there's two possible errors that can happen when we try to open files:\

1. They don't exist.
2. they cannot be opened as plain text files.

The former case can be dealt with easily, as we can write a check for it, using the `FileManager` API:

```swift
mutating func validate() throws {
	if !FileManager.default.fileExists(atPath: filePath) {
		throw ValidationError("'filePath' does not exist")
  }
}
```

And now we can at least show a more user-friendly error when the file does not exist.

But the latter isn't really easy to figure out until you try opening it. You can probably figure out a way to check the first bytes of the file or do anything else crazy before you read the entire file. But it may not be worth it, and in that case, i'd just `throw` the error to the console directly.

The following example will try opening a PDF file I have in my `~/downloads` folder.

```
./CharacterCount /Users/andyibanez/downloads/Formulario.pdf
Error: Error Domain=NSCocoaErrorDomain Code=261 "The file “Formulario.pdf” couldn’t be opened using text encoding Unicode (UTF-8)." UserInfo={NSFilePath=/Users/andyibanez/downloads/Formulario.pdf, NSStringEncoding=4}
```

You could, of course, wrap it inside a `do-catch` block and only print the error when your `catch` is reached. But then, you have to balance. When do I want to show a friendly message? When do I want to show the entire error for the purpose of diagnostics? The good news is that ArgumentParser allows you to deal with errors easily, so you just need to think about whether showing an entire error makes sense or not.

# Conclusion

ArgumentParser lets you validate input and run time exceptions easily. You can write command lines with stricter constrains so your users don't run your tool with weird input.
