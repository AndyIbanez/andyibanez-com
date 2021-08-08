---
title: "Understanding Actors in the New Concurrency Model in Swift"
date: 2021-08-04T07:00:00-04:00
originalDate: 2021-07-29T22:00:31-04:00
publishDate: 2021-08-04T07:00:00-04:00
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
description: "Learn how to use actors in Swift to isolate mutable state and make concurrency safer."
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
6. [Introduction to Unstructured Concurrency in Swift](https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. **Understanding Actors in the New Concurrency Model in Swift**
9. [@MainActor and Global Actors in Swift](/posts/mainactor-and-global-actors-in-swift.md)

<hr>

When we are working with concurrency, the most common problem developers face are data races. Whether it is a task updating a value at the same time another task is reading it or two tasks writing a value so that it it has an invalid value, data races are probably the main pain point of concurrency. Data races are very easy write, and hard to debug. There are entire books dedicated to the problem of data races and established patterns to avoid them.

Data races happen when there's shared mutable state. If you are only working with `let` variables that are never mutated, you are unlikely to encounter them. Unfortunately, even the most trivial of programs does have mutable state at the same point, so racking your brain to make everything immutable is not going to yield results. In general, preferring to use `let` as much as possible and using value semantics (like `struct`s) is going to help a lot when dealing with data races.

Shared mutable state requires synchronization. In the most basic form (which is also the hardest - the case where you are writing all that code yourself), you can make use of locks (a concept that guarantees mutable state will only be modified by one process at a time) and other primitives. In the past few years, many Apple Platform developers have used serial dispatch queues, which are higher level concepts for dealing with concurrency.

Luckily, with Swift 5.5 and the new concurrency APIs introduced at WWDC2021, Swift now has a much easier to way to deal with mutable state, ensuring only one process at a time modifies a value. Of course, this has the same implications as the other new concurrency APIs we have seen in this series so far, which is easy to use, but may be limiting if you need more control. The good news is that the actors API is going to be enough for the vast majority of developers.

# Introducing actors

Actors provide synchronization for mutable state automatically, and they isolate their state from the rest of the program. This means that nobody can modify the shared state unless they go through the actor itself. Because the actor is isolated and you need to talk to it to modify values, the actor ensures that access to its state is mutually exclusive. Only one process will be able to modify its state at a time. Behind the scenes, actors will take care of the manual synchronization for you, and it will "queue up" processes as they attempt to modify it so they only do so one at a time.

## Implementation details

`actors` in Swift are implemented as `actor` types. Similar to how you define `class`es, `enum`s, and `struct`s, you declare an actor by using the `actor` keyword. Actors are reference types, meaning that their behaviors are most similar to `class`es than `struct`s. Which makes complete sense if you think about it, as actors are all about hiding shared *mutable* state that other types may need to access. The main differences between `actor`s and `class`es is that actors implement all the synchronization mechanisms behind the scenes, their data is isolated from the rest of the program, and `actor`s cannot inherit or be inherited from, although they can conform to protocols and be extended.

Thanks to the fact that actors are integrated deeply into the Swift compiler, Swift will do a lot to protect you against code that may run haywire due to its concurrency needs.

Consider the following example:

```swift
class Counter {
    var count = 0
    func increment() -> Int {
        count += 1
        return count
    }
}

class ViewController: UIViewController {
    
    var tasks = [Task<Void, Never>]()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        let counter = Counter()
        
        tasks += [
            Task.detached {
                print(counter.increment())
            }
        ]

        tasks += [
            Task.detached {
                print(counter.increment())
            }
        ]
    }
}
```

*(Apple uses a similar example in the [Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133/) WWDC2021 session)*

*Also, I originally intended to provide a sample Playground with these examples, but I couldn't get it to work as of Xcode 13 Beta 4, so I will provide a standard iOS project instead at the end of this article)*

In this example, you will attempt to increment the counter variable inside [detached tasks](). There is no locking mechanism or any synchronization that ensures that the code will work as you expect it to work. The system could increment to 0 both times, and the values that get printed can be drastically different on each turn.

We can fix it and ensure that the output is always "1, 2" by making `Counter` an `actor` instead of a class.

```swift
actor Counter {
    var count = 0
    func increment() -> Int {
        count += 1
        return count
    }
}
```

Simply doing this change will not be enough. Trying to compile and run it will give you this error in both places where we try to print:

```
Expression is 'async' but is not marked with 'await'
```

This is beautiful, and it really shows you how deeply concurrency is implemented at the compiler level to save you from writing buggy concurrent code, and to save you the time from having to spend hours, days, months, or even years, to learn how to write concurrency safely yourself. I absolutely love the compiler integration, because it also shows you that all the concepts we have explored throughout these series, converge on this point, and we have the compiler helping us make sense of everything we learned so far.

To fix that error, simply add `await` when you call `increment()`.

```swift
print(await counter.increment())
```

The way actors are implemented in Swift is that all public methods are automatically made async for the consumers of your interface. This will allow us to safely interact with actors, because using the `await` keyword will suspend execution until the code is notified that it can go into the actor next and do its job.

*(This is a good point to stop and think if you actually understand `async/await`, which are the most basic building blocks for the new concurrency system in Swift. If you think you need a refresher, you can read the [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/) article of this series.)*

Do note that this has some implications when attempting to access the properties (in this case, `count`) directly. First, you can do read-only access, but it has to be done through asynchronous contexts. This therefore, will not work:

```swift
print(counter.count)
```

It will make the compiler yell you with:

```
Actor-isolated property 'count' can only be referenced from inside the actor
```

This is because, just like methods, properties expose their getters as `async`.

```swift
async {
    let count = await counter.count
    print("count is \(count)")
}
```

Finally, remember when we said nobody can modify the shared state in an actor without going through the actor itself? This means that the actor has to expose methods that would modify its values. You cannot modify properties of an actor directly.

```swift
counter.count = 3
```

```
Actor-isolated property 'count' can only be mutated from inside the actor
```

## Inside the actor

The actor will expose asynchronous code to external callers, helpfully marking everything relevant as `async`. But within the actor itself, all calls are synchronous. This will help you write more natural code within the actor as you won't have to worry about weird execution orders.

You can observe this yourself, add the following method to `Counter`.

```swift
func reset() {
    while count > 0 {
        count -= 1
    }
    print("Done resetting")
}
```

Then, create a new function, `foo`, and start calling `reset()` within it. You will see that the autocomplete suggestions will suggest you autofill with `reset()`.

![Calling reset() within the actor](/img/actors_foo.png)

Whereas, if you call `reset` externally, you will see that the `reset()` method has `async` on its signature.

![Calling reset() outside the actor](/img/actors_reset_async.png).

You can see that anything called within the actor is synchronous (as you can tell due to the lack of the `async` keyword), but calling the very same methods externally are `async`. Synchronous code on the actor *always* runs to completion without being interrupted. You will notice you cannot await on the actor's properties or methods, although nothing prevents the actor from calling async methods from other actors or other places.

# Actor reentrancy

While actors isolate their own state from others, they rarely work alone. They are likely to interact with other actors or with the rest of your codebase in general.

This can cause unexpected behavior. Consider the following example:

```swift
enum ImageDownloadError: Error {
    case badImage
}

func downloadImage(url: URL) async throws -> UIImage {
    let imageRequest = URLRequest(url: url)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}

actor ImageDownloader {
    private var cache: [URL: UIImage] = [:]
    
    func image(from url: URL) async throws -> UIImage {
        if let image = cache[url] {
            return image
        }
        
        let image = try await downloadImage(url: url)
        cache[url] = image
        return image
    }
    
    private func downloadImage(url: URL) async throws -> UIImage {
        let imageRequest = URLRequest(url: url)
        let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
        guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.badImage
        }
        return image
    }
}
```

*(This code is similar to Apple's ImageDownloader code from their [Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133/) WWDC2021 session, but I have created a sample you can run.*

We have an image downloader that caches images so as to not download them again. The `if let` will check if an image is cached and return it if possible. Otherwise the code will download an image, cache it after the download, and return the newly downloaded image. But what happens if we enter here twice?

Consider the following code that uses the `ImageDownloader` actor from above:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    Task.detached {
        await self.downloadImages()
    }
}

//...

func downloadImages() async {
    let downloader = ImageDownloader()
    let imageURL = URL(string:  "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/3.png")!
    async let downloadedImage = downloader.image(from: imageURL)
    async let sameDownloadedImage = downloader.image(from: imageURL)
    var images = [UIImage?]()
    images += [try? await downloadedImage]
    images += [try? await sameDownloadedImage]
}
```

**Important Note:** As of Xcode 13 Beta 4 (and this is an issue that has existed since Beta 1), there is a bug that causes your code to deadlock when entering an actor twice from the same `Task` via `async let`. Apple is aware of this issue, and it will hopefully be fixed in a later beta. The implications of this bug is that the workaround is to use `Task.detached` instead of just `Task` when using more than one `async let` binding at the same time. By the time a later Beta comes out, the GM, or the final release comes out, the bug may be fixed. Please keep that in mind as ultimately, normal `Task`s and `Task.detached` calls have different uses.

We are entering the actor via two different `async let` calls. The first call (`downloadedImage`) will enter the actor and it will execute until it finds the `await` call on `downloadImages`. It will suspend, and the second call, `sameDownloadedImage` will begin executing. Note that `downloadedImage` reached the await, and since it suspended, it hasn't had any time to download the image yet. And because the image is not in the cache, `sameDownloadedImage` will also download the image instead of retrieving it from memory. If you are *really unlucky*, the server may have updated the image behind the same URL, so `downloadedImage` and `sameDownloadedImage` may download different things!

The problem is we are assuming the program state *after* the await call. It's like we are telling the program "Hey, you will download the image, cache it, and anyone else who access it, will grab the cached version". But in reality, it's impossible to make this guarantee with this code, because there may be different calls attempting to access the actor at the same time, and thus we have this bug that hits the network twice for the same image.

To work around this, we can make our actor keep the state of each download, and access that state first-thing before our actor tries to download an image:

```swift
actor ImageDownloader {
    private enum ImageStatus {
        case downloading(_ task: Task<UIImage, Error>)
        case downloaded(_ image: UIImage)
    }
    
    private var cache: [URL: ImageStatus] = [:]
    
    func image(from url: URL) async throws -> UIImage {
        if let imageStatus = cache[url] {
            switch imageStatus {
            case .downloading(let task):
                return try await task.value
            case .downloaded(let image):
                return image
            }
        }
        
        let task = Task {
            try await downloadImage(url: url)
        }
        
        cache[url] = .downloading(task)
        
        do {
            let image = try await task.value
            cache[url] = .downloaded(image)
            return image
        } catch {
            // If an error occurs, we will evict the URL from the cache
            // and rethrow the original error.
            cache.removeValue(forKey: url)
            throw error
        }
    }
    
    private func downloadImage(url: URL) async throws -> UIImage {
        let imageRequest = URLRequest(url: url)
        let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
        guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.badImage
        }
        return image
    }
}
```

*This code is similar to the code provided by Apple in the [Protect mutable state with Swift actors](https://developer.apple.com/wwdc21/10133) WWDC2021 session.*

This looks like a mouthful, but it's very straightforward (and straightforwardness is the power of the new concurrency APIs!). We start by declaring an enum that will hold the state for the current URL. When a URL is downloaded for the first time, we will add this URL to the cache with a `.downloading` status. If any other call is made to the actor with the same URL at the same time, it will see the image is in the cache, so rather than downloading the image again, it will directly `await` on it. Calls made in a farther future will likely see an already downloaded image, so they will return immediately. When the image finishes downloading for the first (and) last time. the image is cached with a `.downloaded` status.

Actor reentrancy prevents deadlocks and guarantees forward progress, but it is necessary that you check your assumptions so as to prevent any other bugs that are not necessarily related to concurrency, such as downloading the same image more than once. Here's a few points to make sure you play with the actor reentrancy concept well:

* Make mutations in synchronous code. You can see that we mutate our cache in the same task, and we are not attempting to update it anywhere else.
* Know that state can change at any point after you hit an `await`. You may need to manually check for some state to determine how it has changed so you can respond to it if necessary.

# Actor isolation

Actors are all about isolation. Their main purpose is to isolate their state from others, so they can manage access to their own properties, ensuring not multiple writes are performed at the same time, leaving your program in a weird state.

Immutable properties can be accessed at any time.

```swift
actor DollMaker {
    let id: Int
    var dolls: [Doll] = []
    
    init(id: Int) {
        self.id = id
    }
}

extension DollMaker: Equatable {
    static func ==(_ lhs: DollMaker, rhs: DollMaker) -> Bool {
        lhs.id == rhs.id
    }
}
```

In the code above, the `==` operator compares two types, and it is a `static` method. `static` means that this method is "outside" of the actor (there's no `self` instance). Combine that with the fact we only access immutable state within the method, and the compiler knows this is a safe thing to do.

```swift
extension DollMaker: Hashable {
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
}
```

On the other hand, this is murky waters. While we also only reference the `id` field, this method is an instance method. It is supposed to be `async` to be isolated. Luckily, in this case, we can explicitly mark the method as `nonisolated` to let the compiler know this is not isolated. The compiler will treat this method as being "outside" the actor, and move on, as long as you only access immutable properties inside of it. If the hasher was using the `dolls` property instead of `id`, this wouldn't work as `dolls` is mutable.

# The Sendable Type

The concurrency model also introduces `Sendable` types. `Sendable` types are those that can be shared concurrently safely. The following are some examples of types that are `Sendable`:

* Value types (such as structs)
* Actor types

Classes can be `Sendable` but only if they are immutable or if they provide their own synchronization within themselves. `Sendable` classes are exceptional.

It is recommended that your concurrent code communicates using `Sendable` types. At some point, Swift will be able to check, at compile time, if you are sharing non `Sendable` types across functions, but this doesn't appear to be the case as of Xcode 14, Beta 4.

## The Sendable Protocol

You probably guessed it, but the way we make types `Sendable` is by making our types conform to the `Sendable` protocol. Just by specifying the conformance, the Swift compiler will do a lot of work for us.

Consider the following example:

```swift
struct Videogame: Sendable {
    var title: String
}

struct VideogameMaker: Sendable {
    var name: String
    var games: [Videogame]
}
```

This will compile without an issue, because `VideogameMaker` is sendable, and so is `Videogame`.

For structs, you can avoid conforming to `Sendable`, and it will still work:

```swift
struct Videogame {
    var title: String
}

struct VideogameMaker: Sendable {
    var name: String
    var games: [Videogame]
}
```

But this is not the case with classes.

```swift
class Videogame {
    var title: String
    
    init(title: String) {
        self.title = title
    }
}

struct VideogameMaker: Sendable {
    var name: String
    var games: [Videogame]
}
```

You will get an error like this:

```
Stored property 'games' of 'Sendable'-conforming struct 'VideogameMaker' has non-sendable type '[Videogame]'
```


## Sendable and generics

A Generic type can be `Sendable` only if its all arguments are `Sendable`.

```swift
struct Pair<T, U> {
    var first: T
    var second: T
}

extension Pair: Sendable where T: Sendable, U: Sendable {}
```

## Sendable functions

For functions that can be passed across actors, they can be made marked as `@Sendable`.

When it comes to closures, marking them as `@Sendable` impose some restrictions. They cannot capture mutable variables from its surrounding scope, everything it captures must be `Sendable`, and finally, they cannot be both asynchronous and actor isolated.

# Conclusion

A sample project for the image download can be downloaded from [here](/archives/Actors.zip).

In this article we explored what actors are and how to use them. We learned that actors isolate their own state and all write access to its properties must be done through the actors. By isolating their own state, actors provide concurrency safety.

We also learned about `Sendable` types and how they are crucial to the new concurrency system in Swift. Sendable types help provide compile-time checks to write concurrent code. As they provide static checking, it's very hard to write incorrect code that breaks the concurrency model or introduces concurrency bugs.