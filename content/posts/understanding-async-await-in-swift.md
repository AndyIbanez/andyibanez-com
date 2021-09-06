---
title: "Understanding async/await in Swift"
date: 2021-06-16T07:01:00-04:00
originalDate: 2021-06-13T00:34:41-04:00
publishDate: 2021-06-16T07:01:00-04:00
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
description: "Learn about Swift's async/await APIs and how to use them, complete with working examples."
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
2. **Understanding async/await in Swift**
3. [Converting closure-based code into async/await in Swift](/posts/converting-closure-based-code-into-async-await-in-swift/)
4. [Structured Concurrency in Swift: Using async let](/posts/structured-concurrency-in-swift-using-async-let/)
5. [Structured Concurrency With Group Tasks in Swift](/posts/structured-concurrency-with-group-tasks-in-swift/)
6. [Introduction to Unstructured Concurrency in Swift](/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. [Understanding Actors in the New Concurrency Model in Swift](/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)
9. [@MainActor and Global Actors in Swift](/posts/mainactor-and-global-actors-in-swift/)
10. [Sharing Data Across Tasks with the @TaskLocal property wrapper in the new Swift Concurrency Model](posts/sharing-data-across-tasks-tasklocal-new-swift-concurrency-model)
11. [Using AsyncSequence in Swift](/posts/using-asyncsequence-in-swift/)
12. [Modern Swift Concurrency Summary, Cheatsheet, and Thanks](/posts/modern-swift-concurrency-summary-cheatsheet-thanks/)

<hr>

Before you try to dive in with concurrency in Swift, you need to understand async/await. There's no way around it. While async/await are not the only [concurrency options](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/), Apple's SDKs are starting to make heavy use of them. There is no doubt that third-party library providers will start offering these as well.

This article will explore async/await and nothing else. Once you understand these concepts, we will start moving on to more advanced articles where we cover structured concurrency, unstructured concurrency, SwiftUI, and more.

If you have been writing callback-based concurrency, keep in mind that the implementation for async/await is *very* different from anything you have seen before in Apple's technologies. It basically throws what you know about concurrent programming out the window. It's important to keep that in mind as you read this article.

In this article, we will write a function that downloads an image and then its metadata using a different network call. We will show you how doing this with callback-based concurrency can become hard to manage quickly, and how async/await solves this problem beautifully.

# Refreshing Concepts

## A Refresher on Procedural Programming

When you write any normal program with no exceptional needs such as networking and/or I/O, your program executes in the order your code is written, calling procedures as needed, and returning content to the caller if necessary.

Consider the following code:

```swift
func sayHi() {
    print("Hi")
}

func multiply(_ x: Int, _ y: Int) -> Int {
    x * y
}

func sayBye(result: Int) {
    print("Bye \(result)")
}

func performCoolStuff() {
    sayHi()
    let x = 10
    let y = 5
    let result = multiply(x, y)
    sayBye(result: result)
}

// Calling performCoolStuff()
performCoolStuff()
```

When you call `performCoolStuff()`, your code is executed as follows:

1. It will first call `sayHi()`
2. It will declare two variables, `x` and `y`.
3. It will call `multiply` passing in the values for `x` and `y`.
4. It will call `sayBye` with the result of the multiplication

There's nowhere to get lost here. Your code is called in the same order it was given. Functions that call other functions are placed in the *call stack* exactly as they appear, unwinding back as they return values to the main callers. As calls happen, the function give back control to the caller through the use of `return`. When we call `multiply`, we assign control to it, and when it returns us a result, it gives us back control through `return`.

You don't think much about procedural programming. Chances are you do it daily, and it always works the way you expect it to \*.

## A refresher on callback-based concurrency code.

Things are a bit more complicated when it comes to code that may run in parallel with other code. Consider the following example that will download an image through a network call and the metadata through a different network call (you can copy and paste this code in a view controller of a new project - it contains everything you need to run it). The download takes place at the same time as the main thread's execution:

```swift
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

func sayHi() {
    print("Hi")
}

func multiply(_ x: Int, _ y: Int) -> Int {
    x * y
}

func sayBye(result: Int) {
    print("Bye \(result)")
}

func downloadImageAndMetadata(
    imageNumber: Int,
    completionHandler: @escaping (_ image: DetailedImage?, _ error: Error?) -> Void
) {
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).png")!
    let imageTask = URLSession.shared.dataTask(with: imageUrl) { data, response, error in
        guard let data = data, let image = UIImage(data: data), (response as? HTTPURLResponse)?.statusCode == 200 else {
            completionHandler(nil, ImageDownloadError.badImage)
            return
        }
        let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).json")!
        let metadataTask = URLSession.shared.dataTask(with: metadataUrl) { data, response, error in
            guard let data = data, let metadata = try? JSONDecoder().decode(ImageMetadata.self, from: data),  (response as? HTTPURLResponse)?.statusCode == 200 else {
                completionHandler(nil, ImageDownloadError.invalidMetadata)
                return
            }
            let detailedImage = DetailedImage(image: image, metadata: metadata)
            completionHandler(detailedImage, nil)
        }
        metadataTask.resume()
    }
    imageTask.resume()
}

func performMessyStuff() {
    sayHi()
    let x = 10
    downloadImageAndMetadata(imageNumber: 1) { image, error in
        DispatchQueue.main.async {
            print("We got results")
        }
    }
    let y = 5
    let result = multiply(x, y)
    sayBye(result: result)
}

performMessyStuff()
```

**Note**: *Apple used a similar example in the [Meet async/await in Swift session](https://developer.apple.com/videos/play/wwdc2021/10132/) at WWDC2021. This example is based on that, but I created a compilable version you can use.*

This is what happens:

1. The method calls `sayHi()` normally.
2. We create a variable `x` and assign it a value.
3. `downloadImageAndMetadata` is called, which internally will set up the first variables it needs for its execution (`imageUrl`).
4. We create a variable, once again synchronously, that will hold a `dataTask` and provide it with a completion handler that will be called after it's done downloading.
5. We call `resume()` on the task to begin the download.
6. The contents of the completion handler will not be executed immediately. Instead, while the downloads happen, the program continues its execution.
7. The program may, or may not, print `"We got results"`. In the case of a network download, it will always take a while, but if this were a faster asynchronous operation, it may be called at this point. The program will create a variable `y`.
8. If both downloads have finished successfully, the program may print `"We got results".` Otherwise it creates the `result` variable and calls `multiply`, which may or may not finish before the downloads.
9. If the downloads have finished successfully, the program will print `"We got results"`, otherwise it will call `sayBye`.
10. Somewhere above there and at any point, the program may start the metadata download task after the image task has downloaded.

This flow of execution is messy, because downloading data from the network is asynchronous and all its work happens somewhere else. Anything else may happen on the main thread while the downloads take place. Whatever the console prints may have a different output on each run \*\*. The downloads spawn from the main thread onto another thread, but the program will continue executing the code in the main thread without any issue. This makes it hard to think procedurally, because we rely on the `completionHandler` to let us know when it has finished its work. If there are tasks that can be performed in the main thread, but they depend on an image and/or its metadata, we have to do all that work in the completion handler (while rerouting the work to the main thread with `DispatchQueue.main.async` whenever relevant).

In the case of callback-based asynchronous code, control is given back whenever completion handlers are executed.

And as you may imagine, these calls can become more and more complex and nested.

# Introducing async/await

If I had to explain async/await in few words, I'd say this:

> async/await is like a hybrid between procedural programming and callback-based closures.

Before we explain why, let's keep two things in mind:

1. Procedural code runs from top to bottom. Control is given back to the caller through `return`
2. Callback-based concurrency will create asynchronous tasks, but it will continue executing the current thread without an issue, even if those tasks are running. Control is given back to the caller through completion handlers.

Let's take a few minutes to discuss the `async` and `await` keywords individually.

## async

`async` has two uses:

* To tell the compiler when a piece of code is asynchronous.
* To spawn asynchronous tasks in parallel.

To mark a function as `async`, simply put the keyword after the function's closing parenthesis and before the arrow, like this:

```swift
func downloadImage(id: Int) async -> UIImage? { ... }
```

Or:

```swift
func downloadImage(id: Int) async throws -> UIImage { ... }
```

You can already see a huge advantage here. The completion handler is gone, and our function signature is very clear with its purpose. We can tell at first glance if it is `async` and what it returns.

`async` code can only run in *concurrent* contexts. That is to say, within other `async` functions, or when manually dispatched via `Task {}`. We will explore `Task {}` in a bit.

## await

`await` is where the magic happens. Whenever the program finds the `await` keywords, it has the option of suspending the function. It may or may not do so, and the decision is up to the system.

If the system does suspend the function, `await` will return control, not to the caller, but to the system. The system will then use the thread to perform other work until the suspended function is done. The statements below `await` will not be executed until it has finished. The system decides what's important to execute, and at some point, it will return control back to you after it sees the `await`ed function has finished.

You can think of it as a traffic light. If you are driving down the road and you find a red light, chances are you will stop. But if it is 4 AM in the morning and there's no cars coming you may just run it. \*\*\*

**What you need to understand about await is that, if it does choose to suspend, nothing below it will execute until the system tells it to, and the system will use the thread to do other work.**

Every call to an `async` function, must be marked as `await`.

To better understand this, we will rewrite our `downloadImageAndMetadata` function, this time using `async` and using `await` within the body.

```swift
    func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {

        // Attempt to download the image first.
        let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).png")!
        let imageRequest = URLRequest(url: imageUrl)
        let (imageData, imageResponse) = try await URLSession.shared.data(for: imageRequest)
        guard let image = UIImage(data: imageData), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.badImage
        }

        // If there were no issues, continue downloading the metadata.
        let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).json")!
        let metadataRequest = URLRequest(url: metadataUrl)
        let (metadataData, metadataResponse) = try await URLSession.shared.data(for: metadataRequest)
        guard (metadataResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.invalidMetadata
        }

        let detailedImage = DetailedImage(image: image, metadata: try JSONDecoder().decode(ImageMetadata.self, from: metadataData))

        return detailedImage
    }
```

This is a long function, but it's already much better than the pyramid version of it. Let's highlight the important parts first:

1. The program procedurally creates `imageUrl` and `imageRequest`.
2. The program reaches a call to an async call, `URLSession.shared.data(for:)`.
3. The program will make a decision on suspending the function or continuing it. In this case, it's likely it will suspend due to the nature of networking, but don't get used to taking that for granted. We will assume the program suspends the function.
4. This will give control back to the system.
5. The system may do other work that is not relevant to this task while the download `await`s.
6. Anything under the first await *will not* be executed. It will not reach the guard, it will not create the variables for the metadata, it will do *nothing* until the `await`ed function finishes.
7. After some time, the system will give control back to you, after the `await`ed function has finished.
8. The `guard` statement is reached, throwing an error if necessary.
9. The program will repeat steps 2-8 but for the metadata task.
10. The program will return a new `DetailedImage`.

As you can see, it is a pretty linear flow, and the way `await` suspends the rest of the execution until the system deems it necessary makes it behave very much like procedural programming.

We can separate that function into different functions as well:

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
```

As long as we mark the functions as `async`, this is possible to do.

It is important to note the linearity of this. The metadata and image **are not being downloaded at the same time**. It will download the image first, and the metadata later. We can make it download both the image and metadata at the same time, but this article is not about actual concurrency just yet. We will explore how to do both tasks at the same time when we learn about *structured concurrency*.

If you want to see the function suspension in action, simply put some print statements before and after `await` code. You will see that the print statements will be executed slowly, as the system suspends the download tasks, perform other tasks, and gives control back to you.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    print("Will download image")
    let image = try await downloadImage(imageNumber: imageNumber)
    print("Has downloaded image")
    print("Will download metadata")
    let metadata = try await downloadMetadata(for: imageNumber)
    print("Has downloaded metadata")
    return DetailedImage(image: image, metadata: metadata)
}
```

If your internet is a bit too speedy to appreciate the slow prints, Apple provides us a neat method: `Task.sleep`. This function solely exists to sleep the thread for a given number of time, and you can use it to explore async/await.

* **Note**: *Unfortunately `Task.sleep` appears to crash as of Xcode 13 Beta 1. `await Task.sleep(2 * 1_000_000_000) `.*

One final important note about `await`: It's not guaranteed that the same thread that executed the code above it is the same one that will execute the code below it (commonly called the *continuation*). This has important implications when dealing with UI. If you use `await` in a context that needs the main thread such as a ViewController, make sure you mark the functions with `await` with the `@MainActor` attribute, or add the attribute to the entire class declaration. If you want a complete tour behind how the new concurrency works in Swift, check out the [Swift concurrency: Behind the scenes](https://developer.apple.com/videos/play/wwdc2021/10254/) WWDC2021 session talk.

## "Bridging" between the sync and async worlds with Task{}

We can create a "bridge" between the sync and async worlds creating a `Task`. To understand why this is necessary, consider the following piece of code:

```swift
func performDownload() {
    let imageDetail = try? await downloadMetadata(for: 1)
}
```

The compiler will protect us from erroneously running this, showing the following error:

> 'async' call in a function that does not support concurrency
> Add 'async' to function 'performDownload()' to make it asynchronous


The compiler is suggesting we mark `performDownload` as async.

```swift
func performDownload() async {
    let imageDetail = try? await downloadMetadata(for: 1)
}
```

But this is not always possible. What if `performDownload` is in a view controller or in another place that can't give you an asynchronous context?

To fix this, we can bridge this synchronous function to the asynchronous world using `Task {}`.

```swift
func performDownload() {
    Task {
        let imageDetail = try? await downloadMetadata(for: 1)
    }
}
```

We are explicitly creating an asynchronous context, and it will behave as such. We can now call perform download from any sync context without an issue.

## get async

To make things even better, properties that are read-only can be `await`ed.

Suppose you have the following wrapper object:

```swift
struct Character {
    let id: Int
}
```

We can get its image and metadata by calling `downloadImageAndMetadata`, but you could also give this object two calculated properties to get its image and/or metadata independently.

```swift
struct Character {
    let id: Int

    var metadata: ImageMetadata {
        get async throws {
            let metadata = try await downloadMetadata(for: id)
            return metadata
        }
    }

    var image: UIImage {
        get async throws {
            return try await downloadImage(imageNumber: id)
        }
    }
}
```

And we can use it as such:

```swift
let metadata = try? await character.metadata
```

# Summary

This was a long introduction to async/await, but hopefully the included examples and discussion will help you understand how this works. `async/await` are the heart of the new concurrency system, so you need to have a fine grasp of them. Future articles may not be as long. Generally, covering the basics of something requires a lot of effort as it's important to not miss any details. Hopefully this article will be of use to you.

I have created a sample project that makes use of the downloaded image and metadata in a UIKit project. You can download it from [here](/archives/AsyncAwaitIntro.zip).

When ran, the program will simply download the contents and display them like this:

![async/await Tutorial 1 result](/img/async_await_part_1.png)

On the `viewDidAppear` method, you will find the following code:

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)

    // MARK: METHOD 1 - Using Async/Await

    Task {
        if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
            self.imageView.image = imageDetail.image
            self.metadata.text = "\(imageDetail.metadata.name) (\(imageDetail.metadata.firstAppearance) - \(imageDetail.metadata.year))"
        }
    }

    // MARK: METHOD 2 - Using async properties

//        Task {
//            let character = Character(id: 1)
//            if
//                let metadata = try? await character.metadata,
//                let image = try? await character.image{
//                imageView.image = image
//                self.metadata.text = "\(metadata.name) (\(metadata.firstAppearance) - \(metadata.year))"
//            }
//        }

    // MARK: Method 3 - Using Callbacks

//        downloadImageAndMetadata(imageNumber: 1) { imageDetail, error in
//            DispatchQueue.main.async {
//                if let imageDetail = imageDetail {
//                    self.imageView.image = imageDetail.image
//                    self.metadata.text =  "\(imageDetail.metadata.name) (\(imageDetail.metadata.firstAppearance) - \(imageDetail.metadata.year))"
//                }
//            }
//        }
}
```

You can comment and uncomment everything under `MARK: - Method x` to fill the outlets with the data provided by the different methods of getting the data. Hopefully you can play around with this to get a better hang of how `async/await` work in Swift.

I want to revisit these two points I made earlier:

> 1. Procedural code runs from top to bottom. Control is given back to the caller through `return`
> 2. Callback-based concurrency will create asynchronous tasks, but it will continue executing the current thread without an issue, even if those tasks are running. Control is given back to the caller through completion handlers.

We can now append one more thing to summarize:

> 3 async/await will run in order just like procedural programming. When it finds an `await` call, the job will  suspend and will give control back to the system instead of the caller. Unlike callback-based concurrency, it will not continue execution of the statements under it until it has finished. The system will make use of the thread to perform other work, and when it decides it's time to revisit your function, it will, and execution will resume linearly.

When you are ready, you can proceed to the third article in the series, [Converting Closure-based code into async/await in Swift](/posts/converting-closure-based-code-into-async-await-in-swift/) to learn more about continuations, explicit continuations, and how to bridge closure-based and delegate-based code into async/await.

### Notes

\*: Well, except when you put bugs in.

\*\*: This is not obvious in this example, but there exists asynchronous code that is much faster than a network call and it may finish much faster than expected, altering the output of the console on each run.

\*\*\*: Drive responsibly.
