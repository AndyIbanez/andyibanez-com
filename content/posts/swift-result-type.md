---
title: "Understanding the Result Type in Swift"
date: 2019-09-11T15:17:02-04:00
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
 - result
 - error
categories:
 - development
description: "Learn to use and understand the Result type in Swift."
keywords:
 - swift
 - result
 - error
 - ios
 - tvos
 - ipados
 - watchos
---

Error handling when expecting a result out of an operation is a very common thing to do. For this reason, various high-level programming languages have introduced a `Result` type into their libraries, on top of their existing error-handling features. This feature was implemented in Swift 5.

A `Result` wraps a success or a failure. It is essentially an `enum` with two possible cases: `.success` and `.failure`. The `.success` case wraps the correct result of an operation, whereas a `.failure` wraps an `Error`. Its implementation uses generics, so you always know what you are going to get back.

In this article, we will explore why you would want to use the `Result` when we can already handle errors with `do-try-catch` and specifying functions that throw an error with `throws`.

# Using Result

To understand `Result` better, we will explore all the ways you could want to write a function to read the contents of a URL into a `String`. On each step, we will see the shortcomings of the traditional `do-try-catch` APIs and the advantages of `Result`. This is a good example to build, because it's a task that is likely to throw an error, and there's different reasons that may trigger it.

## The Traditional Error Handling Approach.

We will start by creating our own error object to handle all file reading operation errors:

```swift
enum StringFileError: Error {
    case invalidURL
}
```

Now, the first attempt to write a file-reading function that can throw errors could look like this:

```swift
func readFileFromURL(url: URL?) throws -> String {
    guard let url = url else { throw StringFileError.invalidURL }
    return try String(contentsOf: url)
}
```

Because the function is marked as `throws`, we have to handle the error ourselves.

```swift
do {
    let fileContents = try readFileFromURL(url: URL(string: "/path/to/file"))
} catch let error as StringFileError {
    // Handle StringFileErrors
} catch {
    // Handle any other errors thrown by the SDK.
}
```

When trying to read a file from a URL, we can receive `StringFileError`s or we can receive `Error`s, depending on what the problem is.

That code gets the job done, but we can improve it with the new `Result` type.

## Implementing Result Over Traditional do-try-catch

To start using result, we can change the signature of the function and its implementation to return a new `Result<String, Error>` instead. It's still far from what we want to achieve, but we will get there in a second.

```swift
func readFileFromURL(url: URL?) -> Result<String, Error> {
    return Result<String, Error> {
        guard let url = url else { throw StringFileError.invalidURL }
        return try String(contentsOf: url)
    }
}
```

To use it, we need to change a few things and even add an extra line of code:

```swift
do {
    let fileContentsResult = readFileFromURL(url: URL(string: "/path/to/file"))
    let fileContents = try fileContentsResult.get()
} catch let error as StringFileError {
    // Handle StringFileErrors
} catch {
    // Handle any other errors thrown by the SDK.
}
```

This is not really a good example of when you'd want to use result. The most important feature of `Result` to me is that you can start using type-safe errors. One of the biggest weaknesses in Swift right now is that errors are not type-safe. Despite how strong-typed Swift is, there is no way to tell if a function can throw specific errors (like `StringFileError`) - everything just `throws` and whatever it throws can be an `Error` of any type, whether it is the high-level `Error` itself or specific implementations.

If you knew that all your file paths are always going to point to a valid file, for example, you can specify the error in result to be of `StringFileError`, and we gain some additional safety.

We can also rewrite the implementation of function, while still using result. Sometimes, `init`int a result type with the closure can be a bit tricky, so we can instead make our function return `.success` or `.failure`.

```swift
func readFileFromURL(url: URL?) -> Result<String, StringFileError> {
    guard let url = url else { return .failure(.invalidURL) }
    return .success(try! String(contentsOf: url))
}
```

To handle the error, we now have just one case to consider, so we can get rid of the last `catch` clause.

```swift
do {
    let fileContentsResult = readFileFromURL(url: URL(string: "/path/to/file"))
    let fileContents = try fileContentsResult.get()
} catch {
    // Now all the possible errors are StringFileErrors, so we can simplify the catch clauses into one
}
```

And you can make it even prettier:

```swift
let fileContentsResult = readFileFromURL(url: URL(string: "/path/to/file"))

switch fileContentsResult {
    case .success(let contents): print(contents)
    case .failure(let stringFileError): // Handle error
}
```

# Conclusion

One of Swift's weaknesses is that it can't deal with errors in a type safe manner despite how strongly-typed it is. The `Result` type helps us write code where errors are typed, and this can help us write cleaner code.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.