---
title: "Unstructured Concurrency With Detached Tasks in Swift"
date: 2021-07-28T07:00:00-04:00
originalDate: 2021-07-14T09:50:44-04:00
publishDate: 2021-07-28T07:00:00-04:00
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
description: "Learn to use detached tasks in Swift for concurrency and why they are useful."
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
5. [Structured Concurrency With Task Groups in Swift](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/)
6. [Introduction to Unstructured Concurrency in Swift](https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/)
7. **Unstructured Concurrency With Detached Tasks in Swift**
8. [Understanding Actors in the New Concurrency Model in Swift](/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)
9. [@MainActor and Global Actors in Swift](/posts/mainactor-and-global-actors-in-swift/)
10. [Sharing Data Across Tasks with the @TaskLocal property wrapper in the new Swift Concurrency Model](posts/sharing-data-across-tasks-tasklocal-new-swift-concurrency-model)
11. [Using AsyncSequence in Swift](/posts/using-asyncsequence-in-swift/)

<hr>

*Understanding async tasks is a requirement to read this article. If you don't understand async tasks, you can read the [Introduction to Unstructured Concurrency in Swift]() article from this Article Series*

Throughout this article series, we have explored what `async/await` is. We have also gotten our feet wet by exploring structured concurrency with `async let` and `Group Tasks`. We have explored that sometimes, structured concurrency, while nice, is not going to cover all our cases, so we mentioned the existence of unstructured concurrency and we have explored how to use `Task {}`  blocks to launch unstructured tasks.

In this article, we will explore the final method to implement unstructured concurrency by using the most flexible method provided to us by Swift 5.5: Detached tasks.

# Introducing Detached Tasks

Out of all the concurrency options we have explored in the new `async/await` system, detached tasks offer the most flexibility. They can be launched from anywhere, their lifetime is not scoped, you can cancel them manually (through a `Task.Handle`) and await them, and they are the one type of tasks that don't inherit anything from the parent tasks. Not even the priority. They are independent from their context.

Detached tasks are useful when you need to perform a task that is completely independent of the parent task. One example is downloading images from the network, and later caching them to disk (this example is used by Apple in the WWDC2021 [Explore Structured Concurrency in Swift](https://developer.apple.com/videos/play/wwdc2021/10134/) talk). The caching operation can happen in a detached task because once we have the image, there is no reason that a cancellation on the download task should cause the caching operation to be cancelled as well.

```swift
func storeImageInDisk(image: UIImage) async {
    guard
        let imageData = image.pngData(),
        let cachesUrl = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask).first else { return }
    let imageUrl = cachesUrl.appendingPathComponent(UUID().uuidString)
    try? imageData.write(to: imageUrl)
}
```

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    Task.detached(priority: .background) {
        await storeImageInDisk(image: image)
    }
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}
```

We have created a `storeImageInDisk` task. Then we call this method within a `Task.detached` in `downloadImageAndMetadata`. Right after the image is downloaded, we will try to cache it.

It's really simple, and once you understand `Task {}`, you can understand `Task.detached {}`. When launching a detached task, you can specify the `priority`. In our case, we used `background`, because it's not a user task that needs to finish with high priority. `.userInitiated` would mean the user cares about that task, and it needs to have high priority.

Because these tasks are unstructured, `Task.detached` will return a `Task` handle we can use to cancel the task at any time. Note that, while `Task.detached` is independent of the task that launched it, all other tasks started within it will still depend on `Task.detached`, so if you cancel a `Task.detached` task, all of its children will be marked as `cancelled`, save for a case in which you run another `Task.detached` within `Task.detached`, and so on.

# Summary

`Task.detached` is not too hard to understand once you understand `Task {}`. They behave almost the same way. The main differences are `Task.detached` will not inherit anything from the parent context. You can cancel both manually. They are great for running non-dependent tasks at any given time.
