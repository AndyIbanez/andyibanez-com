---
title: "A File Download Queue in Combine for Swift"
date: 2020-08-12T07:00:00-04:00
publishDate: 2020-08-12T07:00:00-04:00
originalDate: 2020-08-09T22:13:07-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - macos
 - tvos
 - watchos
 - combine
categories:
 - development
description: "Learn how to use the powerful NSPredicate API for searching and filtering."
keywords:
 - swift
 - combine
 - ios
 - tvos
 - ipados
 - watchos
---

Combine allows us to create pipelines for a lot of tasks. Thanks to the fact it can do work concurrently without leveraging callbacks, it is very easy to build things that would otherwise be very complex.

In this short article, we will build a file download queue that downloads images sequentially. You can use this as the base for more complex queues.

The queue will download an array of images sequentially. If you wanted to support concurrent queues, it would probably be wise to instantiate this publisher as many times as necessary.

# The Download Queue

The download queue will begin with an array of strings with URLs pointing to the images we want to download.

```swift
var subscriptions = Set<AnyCancellable>()

let images = [
    "https://static.zerochan.net/Myne.%28Honzuki.no.Gekokujou%29.full.2884727.jpg",
    "https://i.ytimg.com/vi/-CV-EvHCrwY/maxresdefault.jpg",
    "https://image.tmdb.org/t/p/original/sCabfIegk8pvg7cquPqgWeN72Vo.jpg"
]
```

Arrays in Swift have a `.publisher` property that immediately turns arrays into publishers.

```
images
    .publisher
```

This is an array of strings, not URLs. We need URLs as that's what `NSURLSession` tasks need to work. We can do this with the `.compactMap` operator. This operator will do an operation and discard all nil values.

```swift
images
    .publisher
    .compactMap { URL(string: $0) }
```

At this point, our publisher is emitting non-optional `URLs`.

To actually download the images, we need another publisher. `URLSession` has a method called `dataTaskPublisher` which returns a `NSURLSessionDataTask` wrapped in a publisher. We can use the `.flatMap` operator to convert a publisher into another publisher. Here, we will convert our `URL` publisher into `dataTaskPublisher`.

```swift
images
    .publisher
    .compactMap { URL(string: $0) }
    .flatMap {
        URLSession.shared.dataTaskPublisher(for: $0)
    }
```

When this publisher executes, it will give us a ` URLSession.DataTaskPublisher.Output`. This output contains the `.data` of the content we just downloaded.

We will grab only the non-nil `data` for all the download operations we have received. For this, we can once again leverage `compactMap`.

```swift
images
    .publisher
    .compactMap { URL(string: $0) }
    .flatMap {
        URLSession.shared.dataTaskPublisher(for: $0)
    }
    .compactMap { $0.data }
```

We now need to convert this `data` into an image. To do that, we can use the `init(data:)` initializer of `UIImage`. We will use `compactMap` *again*, because this initializer can return nil.

```swift
images
    .publisher
    .compactMap { URL(string: $0) }
    .flatMap {
        URLSession.shared.dataTaskPublisher(for: $0)
    }
    .compactMap { $0.data }
    .compactMap { UIImage(data: $0) }
```

Finally, we plug in a subscriber so the task can start. We will receive each image sequentially, in the order they appear in the array:

```swift
images
    .publisher
    .compactMap { URL(string: $0) }
    .flatMap {
        URLSession.shared.dataTaskPublisher(for: $0)
    }
    .compactMap { $0.data }
    .compactMap { UIImage(data: $0) }
    .sink(receiveCompletion: ( { state in
    // Handle completion here
    })) { output in
        // Each image will be received here. You can do whatever you want with it.
        // uiImages += [output]
    }
    .store(in: &subscriptions)
```

# Other Considerations

Internet connections can be spotty. For that reason, you could add the `retry` operator in the pipeline, so Combine will try to redownload other failed files as many times a you specify.

If you receive an other somewhere in the pipeline (with incorrect URLs, for example), the completion will be called on your subscription with a failure. This will cancel the entire subscription if the error happens early in. Consider handling the errors with a `.catch` block to deal with the error properly according to the context of your app.

# Conclusion

Combine makes it very easy to create tasks that become too complicated if you try to do them the old way with completion handlers and delegates. Streamlining everything into a pipeline makes it very easy to chain dependent operations without creating many "pyramids of doom".

