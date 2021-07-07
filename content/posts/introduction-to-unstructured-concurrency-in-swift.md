---
title: "Introduction to Unstructured Concurrency in Swift"
date: 2021-07-14T07:00:00-04:00
originalDate: 2021-07-07T09:54:51-04:00
publishDate: 2021-07-14T07:00:00-04:00
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
description: "Learn how to use unstructured concurrency in Swift with the new mechanism, when structured concurrency doesn't suit your needs."
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

###### Table of Contents

1. [Modern Concurrency in Swift: Introduction](/posts/modern-concurrency-in-swift-introduction/)
2. [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)
3. [Converting closure-based code into async/await in Swift](/posts/converting-closure-based-code-into-async-await-in-swift/)
4. [Structured Concurrency in Swift: Using async let](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/)
5. [Structured Concurrency With Group Tasks in Swift](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/)
6. **Introduction to Unstructured Concurrency in Swift**

<hr>

*Understanding Structured Concurrency in Swift is a pre-requisite to read this article. If you aren't familiar with that concept, feel free to read the [Beginning Concurrency in Swift: Structured Concurrency and async-let]() and [Structured Concurrency With Group Tasks in Swift]() articles of this series.*

So far, we have focused in exploring Structured Concurrency with the new APIs introduced in Swift 5.5. Structured Concurrency is great to keep a linear flow in our programs, keeping a hierarchy of tasks that is easy to follow. Structured Concurrency helps a lot with keeping task cancellation in track and making error handling as obvious as it would be with no concurrency. Structured concurrency is a great tool to execute various tasks at once, without having our code run out of control with readability.

# Introducing unstructured concurrency.

Despite the fact that structured concurrency is really useful, there will be times (although hopefully a minority) in which your tasks will have no structured pattern of any kind at all. For these cases, we can leverage unstructured concurrency which will give us more control over the asks, in exchange of some simplicity. The great news is that Swift 5.5 gives us the tools to do this without having to sacrifice a lot of the simplicity. One example of this is giving users the ability to download images, but also giving them the option to cancel the downloads.

There are some situations in which you will feel the need to use unstructured concurrency:

* Launching tasks from non-async contexts. They can outlive their scopes.
* Detached tasks, for tasks that won't inherit any information about their parent task.

In this article, we will focus on the former.

## Launching tasks from non-async contexts

We have actually done this before, and this time we will explain `async {}` blocks in depth. Recall when we began talking about `async/await`, we mentioned that when you need to `await` on a task, you need to be within an `async` context. If you are inside a function that has `async` in the signature, then you are fine, and you can `await` without doing anything special.

The problem is that throughout Apple's SDKs, they weren't designed to support concurrency from the beginning. Take `UIKit` as an example. None of the methods that are part of the lifestyle of a view controller are marked as `async`, such as `viewDidAppear`. If you need to perform concurrency or simply `await` on an `async` task, you can't, unless you use an `async` block.

We actually did this when we talked about [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/). In case you didn't read the original article, by the end of it we ended up with code like this:

```swift
// MARK: - Definitions

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

override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    async {
        if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
            self.imageView.image = imageDetail.image
            self.metadata.text = "\(imageDetail.metadata.name) (\(imageDetail.metadata.firstAppearance) - \(imageDetail.metadata.year))"
        }
    }
}
```

In the code above, we have a few methods that can be `await`ed. We want to call them from `viewDidAppear`, but because `viewDidAppear` does not have `async` as part of the function signature, we can't do so directly. Instead, we need to create an async context using `async`, and we can `await` inside of it.

The implications of doing this are interesting. First, `async {}` actually creates an explicit task. Second, because this launches a new task, anything underneath the `async {}` block will continue executing alongside anything inside the `async {}` block.

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        async {
            if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
                print("Image downloaded")
            }
        }
        
        print("Continue execution alongside the async block")
    }
}
```

If you run this, you will notice that the output is:

```
Continue execution alongside the async block
Image downloaded
```

Executing linear code is much faster than downloading anything from the network, so you are pretty much guaranteed to receive this output every time you run it. It follows to say that, if you have multiple `async {}` blocks, you are launching an async task on each.

Finally (and this is the most interesting part), using `async` this way will actually return you a value of type `Task.Handle<T, E: Error>`. You can later store this handle somewhere and use it to explicitly cancel the task, await its result, and more.

This is where the "unstructured" part comes into play. We can begin a task somewhere, and then we can cancel it from a completely unrelated place.

For example, we can begin the download from the tap of a button:

```swift
// You can get the full code at the end of the article.
class ViewController: UIViewController {
// ...
var downloadAndShowTask: Task.Handle<Void, Never>? {
    didSet {
        if downloadAndShowTask == nil {
            triggerButton.setTitle("Download", for: .normal)
        } else {
            triggerButton.setTitle("Cancel", for: .normal)
        }
    }
}

func downloadAndShowRandomImage() {
    let imageNumber = Int.random(in: 1...3)
    downloadAndShowTask = async {
        do {
            let imageMetadata = try await downloadImageAndMetadata(imageNumber: imageNumber)
            imageView.image = imageMetadata.image
            let metadata = imageMetadata.metadata
            metadataLabel.text = "\(metadata.name) (\(metadata.firstAppearance) - \(metadata.year))"
        } catch {
            showErrorAlert(for: error)
        }
        downloadAndShowTask = nil
    }
}

// Inside ViewController
@IBAction func triggerButtonTouchUpInside(_ sender: Any) {
    if downloadTask == nil {
        // If we have no task going, we have now running task. Therefore, download.
        async {
            await downloadAndShowRandomImage()
        }
    } else {
        // We have a task, let's cancel it.
        cancelDownload()
    }
}

// ...
}
```

And cancel the download when the user wishes to:

```swift
func cancelDownload() {
    downloadAndShowTask?.cancel()
}
```

The full program contains a `triggerButton` whose label changes when `downloadAndShowTask`'s value changes. When it is nil, there's no task going on, so we will use the button to download an image. Otherwise, we will use the button to cancel the action.

`downloadAndShowTask` is of type `Task.Handle<Void, Never>` because the task itself doesn't return anything and it doesn't throw an error. Our button will download the image and set the labels.

If you needed to download the images but not process them directly, you may want to define your tasks in such way that they return specific values.

The following example is more involved, but it shows you the flexibility you have with `async {}` unstructured tasks.

First, we will add `@MainActor` to the declaration of the view controller. It's possible that other threads other than the main one will want to access the values of the view controller.

```swift
@MainActor
class ViewController: UIViewController //...
```

Next, we will change `downloadAndShowTask` to `downloadTask` and we will change the signature to `Task.Handle<DetailedImage, Error>`. This will allow us to `await` on the `DetailedImage`, or to throw an error from within the task if necessary.

```swift
var downloadTask: Task.Handle<DetailedImage, Error>? {
    didSet {
        if downloadTask == nil {
            triggerButton.setTitle("Download", for: .normal)
        } else {
            triggerButton.setTitle("Cancel", for: .normal)
        }
    }
}
```

Next, we will create a new method, ` beginDownloadingRandomImage`, which will start an image download and store it in the `downloadTask` handle. We will create the code that updates the outlets accordingly while we are on it.

```swift
func beginDownloadingRandomImage() {
    let imageNumber = Int.random(in: 1...3)
    downloadTask = async {
        return try await downloadImageAndMetadata(imageNumber: imageNumber)
    }
}

func showImageInfo(imageMetadata: DetailedImage) {
    imageView.image = imageMetadata.image
    let metadata = imageMetadata.metadata
    metadataLabel.text = "\(metadata.name) (\(metadata.firstAppearance) - \(metadata.year))"
}
```

We will update the implementation of `downloadAndShowRandomImage` so it makes use of the two new functions.

```swift
func downloadAndShowRandomImage() async {
    beginDownloadingRandomImage()
    do {
        if let image = try await downloadTask?.get() {
            showImageInfo(imageMetadata: image)
        }
    } catch {
        showErrorAlert(for: error)
    }
    downloadTask = nil
}
```

This method will now call `beginDownloadingImage`, which will assign a value to `downloadTask` within. Then, we call `downloadTask?.get()`. `.get()` will return us the image when it's done downloading, which is why it's `await`ed.

`cancelDownload` is the same as always. We can cancel (and start) the download at any point.

Tasks created this way also inherit the priority, local values, and the actor. They can outlive their scope and therefore give you more control over their lifetimes.

# Summary

We have explored what `async` actually does. We have used it to create an explicit task that we can later cancel manually and explicitly. We can use `async {}` tasks to work with unstructured concurrency that doesn't have any kind of structure. This is useful when we want to have more control over the tasks. Being able to cancel tasks when deemed necessary can help improve the experience of our users, specially if very long-running tasks are involved. These tasks can outlive the original scope they are defined in, enforcing the idea that they create unstructured concurrency.

There is a [a small project](/archives/UnstructuredConcurrencyIntro.zip) with the last pieces of code that you can use to play around and explore to better understand `async {}`. The program has a `Download` button that cancels into a `Cancel` button when a download is in progress. Rapidly tapping the button will show you an alert that says "cancelled" as you cancelled the task explicitly without giving a change to the image to download.

![Download image](/img/async_unstructured_1.png)

![Cancel image download](/img/async_unstructured_2.png)


