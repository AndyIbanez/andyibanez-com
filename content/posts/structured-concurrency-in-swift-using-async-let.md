---
title: "Structured Concurrency in Swift: Using async let"
date: 2021-06-30T07:00:00-04:00
originalDate: 2021-06-23T14:33:48-04:00
publishDate: 2021-06-30T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
categories:
 - article series
 - modern concurrency article series
 - modern concurrency in swift article series
 - development
tags:
 - swift
 - apple
 - programming
 - ios
 - concurrency
 - async
 - await
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - multithreading
description: "Get started using structured concurrency in Swift using async let tasks."
keywords:
 - swift
 - apple
 - programming
 - ios
 - concurrency
 - async
 - await
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - multithreading
---

*This article is part of my [Modern Concurrency in Swift](/posts/modern-concurrency-in-swift-introduction/) article series.*

*This article was originally written creating examples using Xcode 13 beta 1. The article, code samples, and provided sample project have been updated for Xcode 13 beta 3.*

###### Table of Contents

1. [Modern Concurrency in Swift: Introduction](/posts/modern-concurrency-in-swift-introduction/)
2. [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)
3. [Converting closure-based code into async/await in Swift](/posts/converting-closure-based-code-into-async-await-in-swift/)
4. **Structured Concurrency in Swift: Using async let**
5. [Structured Concurrency With Group Tasks in Swift](/posts/structured-concurrency-with-group-tasks-in-swift/)
6. [Introduction to Unstructured Concurrency in Swift](/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. [Understanding Actors in the New Concurrency Model in Swift](/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)
9. [@MainActor and Global Actors in Swift](/posts/mainactor-and-global-actors-in-swift.md)

<hr>

*Understanding async/await is a pre-requisite to read this article. If you aren't familiar with that concept, feel free to read the first part of this article series: [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/).*

`async/await` are the most important concepts when it comes to the new concurrency system in Swift. Understanding async/await will open you the doors to perform multiple tasks in parallel with a clean syntax and straightforward code.

There are actually [multiple ways to do this](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/), but the way Apple has given us at WWDC2021 with Swift 5.5 is the safest one to use, and unless you have highly specific needs, probably the one you will use almost exclusively.

# Introducing Structured Concurrency.

In previous articles, we have discussed how callback-based code can be messy to manage when used in concurrent contexts. For that reason, Apple gave us `async/await`, which is a set of keywords that can help us write concurrent code while keeping a linear flow in our code. This code can be read from top to bottom. However, in [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/), we noted that by just using async/await it doesn't mean that we will perform more than one task at once (the tasks we call may do so internally, though). We will now begin executing some code in parallel, and we will start with the concept of *Structured Concurrency*.

The ideas behind structured concurrency are based on the same ideas as structured programming. We write structured code the vast majority of the time, so you never think about it. Structured code can be read from top to bottom, following a linear flow, in a way that outputs are predictable and code is executed in the exact given order. When using variables, they have a well-defined lifetime within the block they are declared in. In callback-based concurrency, you fire off tasks in different threads or contexts as your main thread keeps executing, creating the potential to alter the output of your program every time it's run. If you are writing Objective-C, you need to treat your variables as `__block` in order to modify them within a block. This creates a labyrinth of code where everything can happen in any order in order to give you the result you want.

Now, consider the following functions:

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}

func downloadImage(imageNumber: Int) async throws -> UIImage {
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).png")!
    let imageRequest = URLRequest(url: imageUrl)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}

func downloadMetadata(for id: Int) async throws -> ImageMetadata {
    let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(id).json")!
    let metadataRequest = URLRequest(url: metadataUrl)
    let (data, metadataResponse) = try await URLSession.shared.data(for: metadataRequest)
    guard (metadataResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.invalidMetadata
    }

    return try JSONDecoder().decode(ImageMetadata.self, from: data)
}

//...

struct ImageMetadata: Codable {
    let name: String
    let firstAppearance: String
    let year: Int
}

struct DetailedImage {
    let image: UIImage
    let metadata: ImageMetadata
}

enum ImageDownloadError: Error {
    case badImage
    case invalidMetadata
}
```

`downloadImageAndMetadata` is a function that will download an image alongside its metadata, all wrapped in a `DetailedImage` object. To perform the download, it will call a `downloadImage` function which will download the image itself,  and a `downloadMetadata` function which will download the metadata. Let's inspect `downloadImageAndMetadata` a little bit deeper.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}
```

The downloads take place sequentially, which is exactly what you want most of the time. The function will download the image first, and the metadata later, one at a time. This is great in many cases, but there are times when the tasks don't have any dependencies with each other and it's therefore possible to execute them concurrently.

In this example, the image download and metadata download are two independent tasks, so we can give the function a little push to download both at the same time and therefore finish earlier.

Before we move on, think about how you would do this with closure-based code. First, you would need to launch to `URLSession` data tasks, each with its own completion handler. But then what? How will the tasks coordinate the completion for this task? What happens if the image downloads first? What happens if the metadata finishes first? How are we gonna "lock" and guarantee access and that the final completion handler is called?

In truth, doing this task with pure closure-based code (and even with delegate-based code), it becomes pretty messy real quick. And we are just talking about a measly *two* tasks at once!

In Swift, we have two ways to work with structured concurrency:

* `async let`
* Task groups

This article will be limited to `async let`, but we will cover Task groups in a future article.

## Understanding tasks

Tasks are the underlying mechanism in which Swift executes your code in parallel. Each task provides a new async context in which it can execute concurrently, alongside other tasks. They will run in parallel automatically as long as it is safe and efficient to do so.

Our `downloadImageAndMetadata` function does not actually create any tasks. Both downloads are `await`ed and this is why they don't run in parallel. We will solve this.

These new concurrency features are deeply integrated into Swift, so as you go along writing concurrent code, the compiler will be there to stop you from introducing common concurrency bugs. I imagine this will be frustrating for new programmers as they will be reported as compiler errors, but in reality, Swift is doing its best to protect you and your code from doing anything crazy. After all, concurrency is a very hard problem to solve. If you have read a book on operating systems you have probably seen that there's multiple patterns developers can make use of in order to write safe concurrent code. But writing this code manually is hard, error prone, and depending on the context, hard to test. Having these checks at compile time is a great security feature.

Marking a function as `async` does not mean a new task will be created - if anything, by default, when the compiler sees a function marked as `async`, it expects it to be `await`ed on each call. Creating tasks is not an automatic process. We can inform the compiler that we want to run concurrent code, but it will be up to it to honor your request. Tasks are always created explicitly.

Structured concurrency is about a balance between simplicity and flexibility. You will be able to do a lot - if not all - of your concurrency work under these constraints, but always remember that if you need even more flexibility, you will find a lower level API that gives you the control you need, but with less safety. Check out my [Multithreading Options on Apple Platforms](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/) article to see an overview of the alternatives.

## Introducing async let

Using `async let`, also called a *concurrent binding*, will launch a task in parallel.

```swift
async let result = //... an async function call (without await)
```

When Swift finds an `async let`, the function to the right side of the equals will begin executing concurrently. That is, where an `await` call would suspend execution of your program there, an `async let` will launch the task but it will continue executing the code underneath it until its value is needed.

Consider the following example:

```swift
func downloadImageConcurrentlyWhilePrinting(imageNumber: Int) async throws -> UIImage {
    print("One lint prints")
    print("We will begin downloading now")
    async let image = downloadImage(imageNumber: imageNumber)
    print("Another line prints until we have the image")
    print("Keep on printing")
    return try await image
}
```

You will notice that all the `print` statements are executed basically instantly. This is because `async let` has launched `downloadImage` as another task. The two `print` statements prior to the `async let` call will be executed as you would expect. The other print statements will print just as quickly because `downloadImage` is not an `await`ed call. By the time we reach `return try await image`, we are telling our program to suspend on the return statement until the image is done downloading (or if an error is thrown).

Because this is one of the mechanisms that will allow us to execute code concurrently, you can have multiple `async let` calls at any given point, and the system will execute them concurrently if possible.

We can now rewrite our `downloadImageAndMetadata` function to download both the image and the metadata at the same time.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}
```

**Note**: *The session most of this article is based on, [Explore structured concurrency in Swift](https://developer.apple.com/videos/play/wwdc2021/10134/), uses a similar example to this, but you can run and play with this one.*

By appending `async` before `let` and moving the `await` keyword to the place where we expect values to exist, we have successfully downloaded multiple things at once, using a structured flow. That's really neat!

And that's it. That's how you can execute code concurrently with the new async/await APIs. This article is not over yet, though. Before we are done, we need to explore a very important concept: The Task Tree.

### The Task Tree

Structured Concurrency makes use of a concept called **The Task Tree**. The task tree is a hierarchy that our structured concurrency code runs on. The task tree influences attributes of our tasks such as cancellation, priority, and local variables. When we jump from one async function to another, the same task is used to execute the new call.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}
```

When we call `downloadImageAndMetadata`, it will inherit all the attributes from the parent task. Each call to `async let` will create a new task - this task is the *child task* of the task the current function is running on.

![async let diagram](/img/async_let_diagram.png)

Our `downloadImageAndMetadata` function can potentially span two child tasks: One for the image, and one for the metadata, and all this code will (again potentially) be running at the same time.

`downloadImageAndMetadata` will inherit the attributes of whatever task it's running on, and `downloadImage` and `downloadMetadata` will in turn inherit the properties from `downloadImageAndMetadata`.

It's important to note that tasks are not children of the *functions* they are running on, although their lifetimes may be tied to them.

The task tree enforces a very important rule: **A parent task can only finish its work as long as all the children have finished their work.**

You can see this enforcement because the `await` calls won't let the execution continue until they are given the green light to continue. Both `downloadImage` and `downloadMetadata` may throw an error or return a value, but in either case, they finish their work before the code that requires them can continue executing.

The normal case for `downloadImageAndMetadata` is that `downloadImage` and `downloadMetadata` will both finish successfully. But what happens if one of them throws an error and the other finishes without a hitch?

The great thing is that you can see, intuitively, and thanks to the fact that the code is *structured* and runs from top to bottom, that whenever one of them throws an error, `downloadImageAndMetadata` will throw the same error. But what happens to the actual execution of the other task? That is, suppose `downloadMetadata` fails and `downloadImage` is downloading a big image. What happens to the image download?

When a task fails, Swift will mark the remaining child tasks as `cancelled`. In this example, since `downloadMetadata` failed, `downloadImage` will be marked as cancelled. Marking a task as `cancelled` does not actually mean that the task is cancelled. Instead, it simply notifies the task that its results are no longer needed. All the child tasks and their descendants will be cancelled when their parent is cancelled.

But when do the tasks actually stop their execution? This is a neat property of structured tasks: cancellation is cooperative. Tasks do not stop immediately. Instead, they will do it as soon as they see it is appropriate. If you have network calls going, it may be inappropriate to just stop them the moment they get the cancel notification.

Tasks have to check for cancellation explicitly. You can check for cancellation from anywhere. This makes it your responsibility to design your code with cancellation in mind, especially if you have tasks that can take a very long time to complete.

There are two ways to check for cancellation. First, you can call `try Task.checkCancellation()` when your functions are marked as `throws`. And second, there is a `Task.isCancelled` which returns a boolean when your tasks are not running inside `throw`ing contexts.

```swift
func downloadImage(imageNumber: Int) async throws -> UIImage {
    try Task.checkCancellation()
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/\(imageNumber).png")!
    let imageRequest = URLRequest(url: imageUrl)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}

func downloadMetadata(for id: Int) async throws -> ImageMetadata {
    try Task.checkCancellation()
    let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/\(id).json")!
    let metadataRequest = URLRequest(url: metadataUrl)
    let (data, metadataResponse) = try await URLSession.shared.data(for: metadataRequest)
    guard (metadataResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.invalidMetadata
    }

    return try JSONDecoder().decode(ImageMetadata.self, from: data)
}

func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}

// NEW FUNCTION
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    var imagesMetadata: [DetailedImage] = []
    for image in images {
        print(image)
        async let image = downloadImageAndMetadata(imageNumber: image)
        imagesMetadata +=  [try await image]
    }
    return imagesMetadata
}
```

In the example above, we have added a cancellation check at the beginning of `downloadImage` and `downloadMetadata`. We have also added a function that will try to download multiple images (although not concurrently - we will learn how perform a variable number of concurrent tasks when we talk about Task Groups). If *any* image or *metadata* download fails, the children tasks will be notified of the cancellation, and if they have a chance to cancel - i.e. if they haven't started downloading their images or metadata - they will stop their execution.

# Summary

We have finally started exploring the world of actual concurrent execution using the new async/await APIs. You learned what structured concurrency is, and a way to implement it with `async let`. You also learned about the task tree and how cancellation is cooperative and how it works.

You may have noticed that our newest function, `downloadMultipleImagesWithMetadata`, will not download all three images at the same time, because it is necessary to `await` the result before we can append it to the array. We will learn how to execute a variable number of concurrent tasks when we begin talking about Task Groups.

In the meantime, take your time to analyze the contents of this article, and as usual, here is [the sample project](/archives/AsyncAwaitConcurrent.zip) you can play around with to better understand the concepts of this article.
