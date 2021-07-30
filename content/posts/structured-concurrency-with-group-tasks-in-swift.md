---
title: "Structured Concurrency With Group Tasks in Swift"
date: 2021-07-07T07:00:00-04:00
originalDate: 2021-06-30T09:51:01-04:00
publishDate: 2021-07-07T07:00:00-04:00
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
description: "Learn about executing a dynamic amount of concurrency in Swift using Group Tasks."
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

*This article is part of my [Modern Concurrency in Swift Article Series](https://www.andyibanez.com/posts/modern-concurrency-in-swift-introduction/).*

*This article was originally written creating examples using Xcode 13 beta 1. The article, code samples, and provided sample project have been updated for Xcode 13 beta 3.*

###### Table of Contents

1. [Modern Concurrency in Swift: Introduction](/posts/modern-concurrency-in-swift-introduction/)
2. [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)
3. [Converting closure-based code into async/await in Swift](/posts/converting-closure-based-code-into-async-await-in-swift/)
4. [Structured Concurrency in Swift: Using async let](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/)
5. **Structured Concurrency With Group Tasks in Swift**
6. [Introduction to Unstructured Concurrency in Swift](/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. [Understanding Actors in the New Concurrency Model in Swift](/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)

<hr>

*Understanding Structured Concurrency and `async let` is a pre-requisite to read this article. If you aren't familiar with that concept, feel free to read the third part of this article series: [Beginning Concurrency in Swift: Structured Concurrency and async-let]().*

Task Groups are the second form of structured concurrency in Swift. When we explored `async let`, we noticed one particular restriction: We cannot run a variable number of tasks at the same time, because when we try to do so, say in a loop, we need to `await` the results. This won't allow us to, for example, download multiple pictures at once, because we are restricted to waiting on each download.

To perform a variable number of tasks, Swift gives us Task Groups.

# Group Tasks

Group Tasks offer more flexibility than `async let` without giving up the simplicity of structured concurrency.

A `Task Group` is a form of structured concurrency designed to provide a dynamic amount of concurrency. With it, we can launch multiple tasks, launch them in a *group*, and have them execute all at the same time.

We have two ways to launch group tasks:

* By calling `withThrowingTaskGroup`
* By calling `withTaskGroup`

Like we have seen multiple times through this article series, you have a variant for tasks that may throw errors and one for tasks that won't. Tasks added to a group cannot outlive the scope of the block in which the group is defined. When child tasks are added to a group, they begin executing immediately and in any order, so take care of designing your code in such a way that there's no dependencies in your child tasks. When the group goes out of scope, the completion of all the tasks within it will be implicitly `await`ed.

Structured concurrency allow us to create `async let` tasks within groups, and the other way around too -, to launch group tasks within `async let` calls. The Task Tree we talked about in the previous article is built upon naturally.

If you try to modify a variable within a task group, like this:

```swift
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    var imagesMetadata: [DetailedImage] = []
    try await withThrowingTaskGroup(of: Void.self, body: { group in
        for image in images {
            group.async {
                async let image = downloadImageAndMetadata(imageNumber: image)
                imagesMetadata +=  [try await image]
            }
        }
    })
    return imagesMetadata
}
```

*This code is a variant of the one written in [Beginning Concurrency in Swift: Structured Concurrency and async-let](). This example is based on the [Explore structured concurrency in Swift](https://developer.apple.com/videos/play/wwdc2021/10134/) WWDC2021 talk.*

The compile will notice that `imagesMetadata` can potentially be accessed unsafely by multiple tasks at the same time. This would lead to data corruption as multiple variables try to write to it at the same time. Luckily, thanks to the fact the new concurrency APIs are deeply integrated into Swift itself, the compiler can do some checks statically and prevent you from introducing such data races.

If you try to compile that, the compiler will yield the following error:

> Mutation of captured var 'imagesMetadata' in concurrently-executing code

So, how exactly can Swift perform these checks?

## The @Sendable closure type

To introduce data race safety, Swift implements the concept of a **@Sendable closure**.

Whenever you create a Task, the body is a `@Sendable` closure, and this closure has the following properties:

* Cannot capture mutable variables.
* You should only capture value types, actors, classes, or other objects that implement their own synchronization. We will explore actors in a future article.

With this knowledge in mind, we can fix our Task Group above. When you create a Task Group with either `withThrowingTaskGroup` or `withTaskGroup`, the task group takes as a parameter the return type your concurrent tasks will create.

```swift
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    try await withThrowingTaskGroup(of: DetailedImage.self, body: { group in
        for image in images {
            group.async {
                async let image = downloadImageAndMetadata(imageNumber: image)
                return try await image
            }
        }
    })
}
```

The implementation of our method is not complete yet, but going step by step, it has received a few important modifications:

* The `of` parameter of `withThrowingTaskGroup` now specifies that it takes `DetailedImage`s.
* Instead of appending to an array, `group.async` will now return an `await`ed `DetailedImage` on each run of the loop.


Essentially, we are "filling the group" with `DetailedImage`s that we will eventually return - unless an error occurs. If an error occurs, the child tasks will be cancelled, and the tasks will need to be stopped. Recall from [Beginning Concurrency in Swift: Structured Concurrency and async-let]() that you are responsible from keeping cancellation in mind when designing your code, but luckily it's a one-line call in the case of structured concurrency.

Our `group` variable is of type `ThrowingTaskGroup<DetailedImage, Error>`. And surprise, this is a collection! You can iterate over it or apply some functional programming into them such as `filter`, `map`, and `reduce`.

```swift
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    var imagesMetadata: [DetailedImage] = []
    try await withThrowingTaskGroup(of: DetailedImage.self, body: { group in
        for image in images {
            group.async {
                async let image = downloadImageAndMetadata(imageNumber: image)
                return try await image
            }
        }
        for try await image in group {
            imagesMetadata += [image]
        }
    })
    return imagesMetadata
}
```

The `for try await` part may throw you off the loop (no pun intended). But alongside all the new concurrency APIs introduced in Swift 5.5, we have a new `AsyncSequence` type. This protocol is implemented by types that will receive values over time. In our `downloadMultipleImagesWithMetadata` function, we use `group.async` to launch a dynamic amount of `DetailedImage` downloads. As the downloads end, they will be delivered in our `for in` loop, one image at a time, making it safe to modify variables within it, and more.

Note that the `await` in the for loop behaves the same way as any other `await` call. Execution will suspend when reaching that point, and when a new image is delivered, execution will continue. This is important to keep in mind because if you have anything underneath the `for-in` loop, it will not be executed until all the elements in the group have been sent through it. If you want to download three images, the three images may download concurrently at the same time, but the for loop will only give you one at a time. If you download a considerable amount of images, they will all have to be downloaded before executing anything underneath `for-in`. This also means that shall an error be thrown, your `for-in` will stop executing (or it may not even execute if the first download fails).

We will explore `AsyncSequence` in depth in a future article.

When we are dealing with Task Groups, we actually have a bit more flexibility. We can for example launch a task asynchronously with a given priority:

```swift
group.async(priority: .userInitiated) {
//...
}
```

Where `priority` is of type [`Task.Priority`](https://developer.apple.com/documentation/swift/task/priority?changes=__6_8), and you have more flexible control when dealing with cancellation, as it even has a `asyncUnlessCancelled` method that can optionally take in the priority as well.

```swift
group.asyncUnlessCancelled(priority: nil) {
   //...
}
```

Finally, you can also call `cancellAll()` in a group. Cancelation will propagate down the tree.

Note that there's a tiny difference when compared to `async let`. When the group goes out of scope through a normal exit, cancelation of the tasks is not implicit. They will just be awaited instead. This is to give your other tasks time to finish and to express the [Fork-Join Model](https://en.wikipedia.org/wiki/Fork–join_model), which is essentially "divide and conquer" - in our case, downloading multiple images in as many child tasks as possible.

# Summary

In this article, you learned the other way to do structured concurrency in Swift by using Task Groups. Task groups allow us to execute dynamic concurrency, such as when needing to download a variable amount of images from a loop. We briefly mentioned `AsyncSequence` and how it's used with Task Groups to deliver results over time.

As it is tradition with this series, [here's](/archives/AsyncAwaitGroupTask.zip) a small example project you can download. While the UI is the same one as the first few projects, you will have a compilable version of `downloadMultipleImagesWithMetadata` that you can play with and experiment.

With this article, we have finished exploring the two methods to do structured concurrency in Swift:

* When you need basic concurrency support, use `async let`.
* When you have a dynamic amount of concurrent tasks to perform, use a Task Group.

In the next article, we will begin exploring unstructured concurrency. It sounds intimidating, but Swift makes it possible to work with as easily as with structured concurrency.
