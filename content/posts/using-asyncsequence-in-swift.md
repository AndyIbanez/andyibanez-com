---
title: "Using AsyncSequence in Swift"
date: 2021-09-01T07:00:00-04:00
draft: false
originalDate: 2021-08-29T22:57:57-04:00
publishDate: 2021-09-01T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
categories:
 - article series
 - modern concurrency article series
 - modern concurrency in swift article series
 - development
tags:
 - asyncstream
 - asyncsequence
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
description: "Learn about AsyncSequence and AsyncStream in Swift, and how to use them."
keywords:
 - asyncstream
 - asyncsequence
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
5. [Structured Concurrency With Group Tasks in Swift](/posts/structured-concurrency-with-group-tasks-in-swift/)
6. [Introduction to Unstructured Concurrency in Swift](/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. [Understanding Actors in the New Concurrency Model in Swift](/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)
9. [@MainActor and Global Actors in Swift](/posts/mainactor-and-global-actors-in-swift/)
10. [Sharing Data Across Tasks with the @TaskLocal property wrapper in the new Swift Concurrency Model](posts/sharing-data-across-tasks-tasklocal-new-swift-concurrency-model)
11. **Using AsyncSequence in Swift**
12. [Modern Swift Concurrency Summary, Cheatsheet, and Thanks](/posts/modern-swift-concurrency-summary-cheatsheet-thanks/)

<hr>

Along the new concurrency APIs introduced in Swift at WWDC2021, we have AsyncSequence. `AsyncSequence` is a collection protocol that allow us to receive data in loops and even top higher order functions - such as `filter`, `map` and `reduce` - asynchronously, being able to `await` for new data as it becomes available.

## Introducing AsyncSequence

As a sequence, we can do with them anything we can do with any other sequences. Other than applying higher order functions, we can also search through them, count the number of elements, and more.

What we need to understand is the underlying behavior of these sequences.

Recall that `await` means suspension - when our code runs, if it encounters an `await` call, it will begin doing the awaited work somewhere else, and execution of your code will stop. When the asynchronous task is done downloading, the compiler will, at some point begin executing everything below the `await`ed call.

`AsyncSequence` has essentially the same behavior, save a key difference.

Imagine you have the following file in a remote server:

```
// videogames.csv
The Legend of Zelda: Ocarina of Time|1998|10
The Legend of Zelda: Majora's Mask|2000|10
The Legend of Zelda: The Wind Waker|2003|10
Tales of Vesperia|2008|8
Tales of Graces|2011|9
Tales of the Abyss|2006|10
Tales of Xillia|2013|10
```

For your convenience, you can find that file [here](https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part11/videogames.csv).

## Using AsyncSequence

It is very easy to consume this file, line by line:

```swift
struct Videogame {
    let title: String
    let year: Int?
    let score: Int?
    
    init(rawLine: String) {
        let splat = rawLine.split(separator: "|")
        self.title = String(splat[0])
        self.year = Int(splat[1])
        self.score = Int(splat[2])
    }
}

//...

func loadVideogames() async {
    let url = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part11/videogames.csv")!
    
    var videogames: [Videogame] = []
    
    do {
        for try await rawVg in url.lines {
            if rawVg.contains("|") {
                // Valid videogame
                videogames += [Videogame(rawLine: rawVg)]
            }
        }
    } catch {
        // Handle the error
    }
}
```

`lines` is an `AsyncSequence` - as the URL obtains new lines from the file, they processed, one by one. It's not accurate to say it's an array or any other kind of specific collection. This really is just an abstraction for something that will deliver values to us overtime. We can also create our own `AsyncSequence`s.

But `AsyncSequence` wouldn't be half as interesting if we could't refactor the code into something more sensible that makes equal sense.

```swift
func loadVideogames() async {
    let url = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part11/videogames.csv")!
    
    let videogames =
        url
        .lines
        .filter { $0.contains("|") }
        .map { Videogame(rawLine: $0) }
    
    do {
        for try await videogame in videogames {
            print("\(videogame.title) (\(videogame.year ?? 0))")
        }
    } catch {
        
    }
}
```

At this point, there's one thing worth mentioning: When we are using `AsyncSequence` this way - chaining multiple calls to transform our "collection -, you will notice that sequence doesn't "start" automatically. If you do not add the `for` loop there, the sequence will not start, and you will see nothing. This means that we have some limitations, like we can't get the number of elements by calling `.count` on `videogames`. I have also noticed that it's missing some methods that you may have seen elsewhere, like `dropLast()`.

The sequence will be `await`ed when it's needed for it to produce a new value - that is, in our specific example, each new videogame newline will trigger an `await`. Each time a new value is emitted, our code is suspended, and the thread is off to do different work, until it either produces a new value, its done, or an error is thrown.

And because this is just a normal iteration, you can use `break` and `continue` within the loop.

```swift
for try await videogame in videogames {
    if videogame.score == 10 {
        continue
    }
    print("\(videogame.title) (\(videogame.year ?? 0))")
}
```

In this example we are adding a `continue` statement to avoid printing all the games with a perfect score. Of course, you could just alternatively add a filter to `videogames` adding this constraint, and all videogames with a score of 10 would not get printed.

```swift
let videogames =
    url
    .lines
    .filter { $0.contains("|") }
    .map { Videogame(rawLine: $0) }
    .filter { $0.score != 10 } // Apply the filter here

do {
    for try await videogame in videogames {
        print("\(videogame.title) (\(videogame.year ?? 0))")
    }
} catch {
    
}
```

One other thing of interest is that in this particular case we are using an `AsyncSequence` that is delivering data over the network. It is also possible to use it with local files.

Apple has added multiple APIs that make use of `AsyncSequence` throughout the SDK, including but not limited to:

* `FileHandle.standardInput.bytes.lines`, which can be used to receive input from the command line or other sources.
*  URLs can access both `lines` and `bytes`, when you want to read an input-as is rather than line by line, by calling `URL`'s `resourceBytes` property.
*  `URLSession` has a `bytes(from:)` method, which you can use to download data byte by byte from the network.
*  `NotificationCenter` now has APIs to `await` on new notifications of the specified types. I may write an article on this eventually.

## Using AsyncStream

It's possible that already have code that continuously delivers updates on certain events via a callback or even delegates. For example, if you are using `CoreLocation` to receive the user's location in realtime, you have code that receives new location points as they become available.

We can streamline code like that - which delivers its results in many different places at once - using an `AsyncStream`. Similar to [Converting closure-based code into async/await in Swift](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/), we can convert "real-time" or "streaming" code into into a sensible async sequence.

To show you this, we will first create a small wrapper for the CoreLocation delegate methods that receive events. This will be a beautiful example, because we will both create a continuation for the authorization status, and then we will setup a stream for the location events.

```swift
@MainActor
class LocationUpdater: NSObject, CLLocationManagerDelegate {
    private(set) var authorizationStatus: CLAuthorizationStatus
    
    private let locationManager: CLLocationManager
    
    // The continuation we will use to asynchronously ask the user permission to track their location.
    private var permissionContinuation: CheckedContinuation<CLAuthorizationStatus, Never>?
    
    var locationHandler: ([CLLocation]) -> Void = { _ in }
    
    override init() {
        locationManager = CLLocationManager()
        authorizationStatus = locationManager.authorizationStatus
        
        super.init()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
    }
    
    func start() {
        locationManager.startUpdatingLocation()
    }
    
    func stop() {
        locationManager.stopUpdatingLocation()
    }
    
    func requestPermission() async -> CLAuthorizationStatus {
        locationManager.requestWhenInUseAuthorization()
        return await withCheckedContinuation { continuation in
            permissionContinuation = continuation
        }
    }
    
    // MARK: - Location Delegate
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        locationHandler(locations)
    }

    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        authorizationStatus = manager.authorizationStatus
        permissionContinuation?.resume(returning: authorizationStatus)
    }
}
```

This `LocationUpdater` class allow users to ask for authorization and it does so with `async` await, thanks to the `permissionContinuation` continuation. Developers can then call this code as follows:

```swift
let authorizationsStatus = await updater.requestPermission()
```

This code will return the status on a single line, even though internally it jumps through two different methods to get the result. If you don't remember or don't know how continuations work, check out my [Converting closure-based code into async/await in Swift](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/) article.

The `var locationHandler: ([CLLocation]) -> Void = { _ in }` property is a closure that will allow us to receive location events without having to implement more delegates on our side. We can wrap this is in a `AsyncStream` and start receiving location events as they happen, and receive them in a loop, and even use the sequence functions to mutate this array later on:

```swift
func beginTracking() async {
    await requestPermission()
    if authorizationsStatus == .authorizedWhenInUse {
        for await location in locationEvents() {
            print(location.speed)
        }
    }
}

func locationEvents() -> AsyncStream<CLLocation> {
    let locations = AsyncStream(CLLocation.self) { continuation in
        updater.locationHandler = { locations in
            locations.forEach {
                continuation.yield($0)
            }
        }
        updater.start()
    }
    return locations
}
```

`locationEvents` is our `AsyncSequence`.

One important note here is that you can listen to the continuation to learn when it is stopped. If you have a sequence that needs to be manually stopped or you need to do some sort of cleanup after receiving events, it is useful to implement. That method is

```swift
continuation.onTermination = { _ in}
```

Unfortunately, implementing that method requires our streaming type - in this case `CLLocation` - to be `@Sendable`. Because CLLocation is not sendable, we cannot use it here. To learn about `@Sendable`, check out the "**The Sendable Type**" section of my [Understanding Actors in the New Concurrency Model in Swift](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/) article. I tried to work around this by creating a wrapper type with a single property `location` property, but it didn't work. At this time, I am not sure what would be the best way to use `AsyncStream` with CoreLocation, other than creating structs with all the same properties as `CLLocation`, which would take a while.

# Conclusion

`AsyncSequence` allows us to await on events as they happen in real time. Whether it is network events or other system events, `AsyncSequence` can help us streamline our code to be easier to read and write. `AsyncStream` can be used to wrap a continuous event emitter into an `AsyncSequence` that can receive its events in a loop, and we can filter, map, reduce, and perform more standard collection operations on them.