---
title: "Modern Swift Concurrency Summary, Cheatsheet, and Thanks"
date: 2021-09-08T07:00:00-04:00
draft: false
originalDate: 2021-09-05T22:51:29-04:00
publishDate: 2021-09-08T07:00:00-04:00
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
description: "A summary of all the articles written about the modern Swift Concurrency System introduced at WWDC2021."
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
11. [Using AsyncSequence in Swift](/posts/using-asyncsequence-in-swift/)
12. **Modern Swift Concurrency Summary, Cheatsheet, and Thanks**

<hr>

Since WWDC21, we have talked, extensively, about all the new concurrency features introduced in Swift 5.5. We covered *a lot* of topics, so I decided to finish off this series writing a summary article were we cover the most important topics of each article. Links will be given to the relevant articles when necessary in case this summary is not enough.

# async/await

* `async` and `await` are the most fundamental keywords of the new concurrency system.
* When you learn to program, you are used to writing code that executes linearly (called procedural programming). Your code executes its lines in the order you give them.
* When dealing with concurrent tasks, prior to async/await, Apple gave us callback/closure-based concurrency, in which we are notified via a closure when the task is done, or delegate-based concurrency in the case of older concurrent code. Callback and delegate based concurrency can alter the order in which our program runs. If we have a set of linear instructions, we can be notified as that linear code executes and receive new data from a different thread. This makes it possible to deal with concurrency, but it can become hard to understand as time goes by.
* async/await allow us to write linear concurrent code that executes from top to bottom. To work with this, functions that can be called asynchronously should be marked as `async` in the function signature.

```swift
func downloadData() async throws -> CustomData { 
	//...
}
```

* When we call a function marked as `async`, it needs to be prepended with the word `await`.

```swift
func processData() async throws -> CustomData {
   let newData = try await downloadData()
   return newData
}
```

* When execution of our code reaches the `await` keyword, execution of our code may be suspended **suspended**, and the thread our code is running on is free to do other work. This other work is assigned by the system. Because our code is suspended, the lines below the `await` will not be executed until the `async` task is done running.
* If our code was suspended, at some point, the `async` task will be done running. The system will come back to our code and continue executing our code. That means everything under the `await` call will resume execution.
* `async/await` allow us to keep a procedural flow that runs from top to bottom thanks to thread suspension.
* Anything below an `await` call is called a *continuation*. This is relevant to know if you want to convert delegate or closure based concurrent code into async/await.
* It's important to note that a continuation may not be executed in the same thread that was suspended. If you need to update your UI, you should run that code in the @MainActor.
* `async` code needs to run in `async` contexts. This means functions marked as `async`, or if you create such context yourself with `Task {}`.

To learn about async/await, checkout the [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/) article.

# Converting Delegate and Closure Based Code into async/await.

* The compiler can already do this for you for free without any effort - if you start typing a method that you expect it to have a closure, you may find the compiler has already created an `async` version of it for you for free.
* You can create such conversions yourself.
* To create such conversions, you create manual *continuations*. Recall a continuation is everything that happens after an `await` call.
* To create these conversions yourself, you can use the `withCheckedContinuation` or `withCheckedThrowingContinuation` functions. Use them to wrap your closure-based calls, or store references to the continuations in order to call them later as part of delegate-based calls.
* These methods will provide you with the continuation you need to call explicitly when your concurrent tasks are done. You can call them passing in the "returned" value, or throwing an error (in the case of `withCheckedThrowingContinuation`.
* You are required to call a continuation exactly once. Don't forget to call it. Do call it once and no more.
* The following code hows how to convert closure-based concurrency into async/await.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    return try await withCheckedThrowingContinuation({
        (continuation: CheckedContinuation<DetailedImage, Error>) in
        downloadImageAndMetadata(imageNumber: imageNumber) { image, error in
            if let image = image {
                continuation.resume(returning: image)
            } else {
                continuation.resume(throwing: error!)
            }
        }
    })
}
```
* Converting delegate-based calls into async/await is slightly more involved, but not impossible. You need to store the continuation provided by the `withChecked*Continuation` call and call it whenever it is appropriate.

```swift
class ContactPicker: NSObject, CNContactPickerDelegate {
    private typealias ContactCheckedContinuation = CheckedContinuation<CNContact, Never> // 1

    private unowned var viewController: UIViewController
    private var contactContinuation: ContactCheckedContinuation? // 2
    private var picker: CNContactPickerViewController

    init(viewController: UIViewController) {
        self.viewController = viewController
        picker = CNContactPickerViewController()
        super.init()
        picker.delegate = self
    }

    func pickContact() async -> CNContact { // 3
        viewController.present(picker, animated: true)
        return await withCheckedContinuation({ (continuation: ContactCheckedContinuation) in
            self.contactContinuation = continuation
        })
    }

    func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
        contactContinuation?.resume(returning: contact) // 4
        contactContinuation = nil
        picker.dismiss(animated: true, completion: nil)
    }
}
```
* You are not limited to converting delegate-based concurrency. Even delegate-based call that does everything in the same thread can benefit from this (but do consider if your effort will be worth it and it won't be over-engineering.

To learn more about converting existing closure or delegate based code into `async/await`, check out the [Converting closure-based code into async/await in Swift](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/) article.

# Structured Concurrency
* Having multiple `await` calls in a row does not mean that concurrency is taking place. The code below is not concurrent, although the `await` calls are independent so they could very well be.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}
```

* Structured concurrency allows us to write concurrent code that can also be read from top to bottom. We can launch multiple tasks in parallel easily.
* There's two types of structured concurrency: `async let` calls and Task Groups.

## async let concurrency

* Calls that can be `await`ed can also be executed concurrently.
* To do so, simply add the `async` keyword in your variable definition before the `let` or `var`, and remove the `await` call.
* Then simply `await` for the variable at the point that you need it.
* The following code is the same code as above, but it now performs both `async` tasks concurrently.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}
```

* Despite the fact that `image` and `metadata` are async values, the code is still very easy to read, because we await for their values before we return from the function.
* `async let` is perfect when you know the exact number of concurrent tasks you need to perform. In the example above, we know we have two: `downloadImage` and `downloadMetadata`.

To learn more about structured concurrency with `async let`, read the [Structured Concurrency in Swit: Using async let](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/) article.

## Group Tasks

* Use group tasks when the amount of concurrency is not known before hand. Like fetching a variable number of URLs from a web service, which you later want to download concurrently.
* To launch them, use the `withThrowingTaskGroup` or `withTaskGroup` methods.
* In the example above, we create a Task Group to download a variable number of images.

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

* the `group` variable will have our data as it is downloaded. It's an `AsyncSequence`, so you can iterate over it, or apply functions such as `filter`, `map`, `reduce`.
* You can specify the priority of the group, making this structured concurrency method a bit more flexible than `async let`.

```swift
group.async(priority: .userInitiated) {
   //...
}
```

* We can be ready for cancellation with `asyncUnlessCancelled`.

```swift
group.asyncUnlessCancelled(priority: nil) {
   //...
}
```

### Sendable Types

* Sendable types are the types that work well with concurrency. The compiler won't complain if you use these in concurrent contexts. There are `@Sendable` closures that only work with `Sendable` types (a protocol).
* `@Sendable` closures cannot capture mutating variables.
* You should only capture value types, actors, classes, or other objects that implement their own synchronization.

To learn more about Group Tasks and/or Sendable types, read the [Structured Concurrency With Task Groups in Swift](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/) and [Understanding Actors in the New Concurrency Model in Swift](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/) articles.

## The Task Tree

An important concept of structured concurrency (for both `async let` and Task Groups) is the Task Tree.
* `async` functions can spawn other `async` tasks. These spawned tasks are the *children* of the task that launched them.
* Children tasks inherit info from their parents, such as priority, local variables, and cancellation.
* It is said that a Parent task can only finish their work when their children have also finished their work.
* Cancellation of a task is governed by the task tree and it is cooperative. When a task is cancelled - either manually via a `cancel` or `cancellAll` call, or when they throw an error -, the tasks in the tree are not cancelled instantly. Instead, the task is marked as `cancelled`, but they continue doing their work until they see it is appropriate to be cancelled. When a parent task is marked as cancelled, its children tasks are marked as `cancelled` as well.
* To check the cancellation status of a task and determine if you need to stop working, you can use the `Task.checkCancellation()` method for tasks that may throw errors, or `Task.isCancelled` for tasks that don't throw.

```swift
func downloadImage(imageNumber: Int) async throws -> UIImage {
    try Task.checkCancellation() // <- If we are cancelled, this throws.
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/\(imageNumber).png")!
    let imageRequest = URLRequest(url: imageUrl)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}
```

To learn more about the Task Tree, check out the [Structured Concurrency in Swit: Using async let](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/) article.

# Unstructured Concurrency

Unstructured concurrency is useful when you don't have such a procedural flow for your tasks, but they can still help you greatly reduce the amount of unusual execution flows. Unstructured concurrency also gives you more control over structured concurrency.

There's two ways to do unstructured concurrency: `Task` calls and detached tasks with `Task.detached`.

## Task {}

* When you use `Task {}`, you are actually launching a concurrent task. This is how the "bridge" between the async and sync worlds is done.
* You can store them in variables so you can manually cancel them when necessary.
* You can also start them with a specific priority.

To learn more about Unstructured Concurrency with tasks, check out the [Introduction to Unstructured Concurrency in Swift](https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/) article.

## Detached tasks

* Launched with `Task.detached {}`.
* Unlike the other kinds of tasks, they do not inherit anything from their parent tasks. Not even the priority.
* They are independent from the context they are launched in.

To learn more about Detached Tasks, check out the [Unstructured Concurrency With Detached Tasks in Swift](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/) article.

# Actors

* Actors are reference types that isolate their state from the rest of the program. This is a perfect mechanism to prevent data races in your program.
* They provide their own internal synchronization for when they are accessed. This prevents data races.
* You cannot modify an actor state directly. Every call that modifies the actor needs to go through the actor itself.
* All the methods the actor provides are exposed through `await` calls even when you don't explicitly mark them as such.
* Properties are method that don't need to be or can't be isolated can be marked as `nonisolated`.
* You should take care to design for actor reentrancy (entering the actor multiple times). Because its state changes, you may need to do some considerations. For example, an actor that downloads and caches images may download and cache the same image twice if entered in quick succession.

[Understanding Actors in the New Concurrency Model in Swift](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)

# @MainActor and Global Actors

* We can define global actors across different files and types. Marking a class as running on a specific actor ensures that all code will be run in the same thread.
* You declare global actors with the `@globalActor` attribute, and then you use that actor by referencing its name prepending a `@`. In the example above, we create an actor called `MediaActor` and we create a variable called `videogames` that runs on this actor.

```swift
@globalActor
struct MediaActor {
  actor ActorType { }

  static let shared: ActorType = ActorType()
}

struct Videogame {
    let id = UUID()
    let name: String
    let releaseYear: Int
    let developer: String
}

@MediaActor var videogames: [Videogame] = []
```

* The `@MainActor` is a special global actor provided by Swift that runs on the main thread. We can mark view controllers, view models, and other code that we want to force to run on the main thread as `@MainActor`. Marking a class with an actor means that all its properties and methods will be run on that same actor. In the example below, we add the `@MainActor` attribute to a view controller, ensure all its code runs on the main thread.

```swift
@MainActor
class GameLibraryViewController: UIViewController {
	//...
	nonisolated var fetchVideogameTypes() -> [VideogameType] { ... }
	//...
}
```

It's possible to override the actor of specific methods.

```swift
@MainActor
class GameLibraryViewController: UIViewController {
   @MediaActor func doThisInAnotherActor() {}
}
```

[@MainActor and Global Actors in Swift](https://www.andyibanez.com/posts/mainactor-and-global-actors-in-swift/)

# Sharing Data Across Tasks with @TaskLocal

* The `@TaskLocal` property wrapper can be used to share data across local tasks.
* The tasks should be part of the same tree - detached tasks launched within some task will not inherit them.

```swift
class ViewController: UIViewController {
    @TaskLocal static var currentVideogame: Videogame?
    // ...
}
```

* Only static properties can have this property wrapper.
* To write values to them, we need to bind them values.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
    
    let vg = Videogame(title: "The Legend of Zelda: Ocarina of Time", year: 1998)
    Self.$currentVideogame.withValue(vg) {
        // we cam launch some async tasks here that make use of the LocalValue
    }
}
```

* Reading them is an `await`ed call.

```swift
func expensiveVidegameOperation() async {
    if let vg = await ViewController.currentVideogame {
        print("We are processing \(vg.title)")
    }
}
```

[Sharing Data Across Tasks with the @TaskLocal property wrapper in the new Swift Concurrency Model](https://www.andyibanez.com/posts/sharing-data-across-tasks-tasklocal-new-swift-concurrency-model/)

# AsyncSequence and AsyncStream

* `AsyncSequence` allows us to receive values over time, `await`ing them in a loop, or even applying functions such as `filter`, `map`, `reduce` to them.

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

* Worth noting that the sequence will not "start" until we put it in a loop. Applying higher order functions simply limits what will be received in the `await for` loop.
* Multiple APIs have been updated to support this in WWDC21, including the `NSNotificationCenter` APIs.
* The `AsyncStream` object can be used to take a stream of values from somewhere and convert it into something that can be used in a `for await` loop.
* For example, if you receive GPS updates in real time in a delegate, you can wrap all that up and receive the new coordinates in a loop instead.

[Using AsyncSequence in Swift](https://www.andyibanez.com/posts/using-asyncsequence-in-swift/)

# Credits and Thanks

The articles in this series have quickly become one of my most visited pages on my website since I relaunched it in 2019. Because of that, I have also received a lot feedback from members in the community.

I want to take a minute to thank everyone who has written me regarding typos or weird phrasing in some sentences. I have taken a lot of care into improving the article with all your opinions and comments. You have all helped me increase the quality of these articles a lot.

I have received a lot of emails, and it's really, really hard to name all of you due to the sheer amount of people who wrote to me. So, thank you all so much for helping me improve the quality of my blog.

There is one person in particular I want to mention by name, because he has spent a lot of time going through all the articles in the series and sending very detailed emails with observations and improvements. This person's emails were actually very long, and whenever I received an email from him, I spent a long time working through the fixes. That said, every second I spent working on his recommendations paid off, and this article series is probably one of the things I'm very proud of. This person is [Dennis Birch](https://twitter.com/dennisbirch2). Big thanks to Dennis for helping this article series become one of my favorites.

I also want to thank to everyone who has shared these article or mentioning them in their own content. I want to thank [Steward Lynch](https://twitter.com/StewartLynch) for the exposure to his YouTube viewers.

And a big thanks to [Dave Verwer](https://twitter.com/daveverwer). His iOSDevWeekly newsletter has not only helped me exposing this series to the world, but also with a lot of articles that I have rewritten since I relaunched my site in 2019.