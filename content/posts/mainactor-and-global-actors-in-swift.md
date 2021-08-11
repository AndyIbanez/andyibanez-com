---
title: "@MainActor and Global Actors in Swift"
date: 2021-08-11T07:00:00-04:00
originalDate: 2021-08-08T19:44:34-04:00
publishDate: 2021-08-11T07:00:00-04:00
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
description: "Learn what @MainActor is and how you can use Global Actors in Swift."
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
5. [Structured Concurrency With Task Groups in Swift](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/)
6. [Introduction to Unstructured Concurrency in Swift](https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. [Understanding Actors in the New Concurrency Model in Swift](/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)
9. **@MainActor and Global Actors in Swift**

<hr>

We have recently talked about [actors](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/), what they are, and how to use them. If you remember, an actor controls access to its properties, so a member doesn't get written from different processes at the same time, avoiding corrupted data.

# It's all about the main thread.

Whether you have been programming for Apple platforms for a while, or you are fairly new, chances are you have heard about the Main Thread. The Main Thread is in charge of running your UI Code. On Apple platforms, we are not allowed to update our UI in any other place outside the main thread. When we are running processes that are commonly asynchronous, it's possible they will return their values on whatever thread they are running on, but we need to deliver those results the main thread. Before the modern concurrency system, we could simply call `DispatchQueue.main.async` and pass in a completion block. This block would run on `main`, making it safe to update our UI from there. Of course, this doesn't mean that we should try to do everything on the main thread, because if the main thready is really busy, it will result in visible performance issues for your users, and if the app becomes unresponsive, the system will kill it after a defined amount of time.

Because the new concurrency system may jump around different threads, suspending tasks, resuming others (which may do so in different threads), and so on, we need another mechanism to update our main thread. This mechanism exists, and it is a "special" kind of actor called the `@MainActor`.

## Introducing the main actor

The main actor, written as `@MainActor`, represents your main thread. The main actor will perform all its synchronization on the main dispatch queue. This actor is "special" because it can be found all over Apple's frameworks. It's on SwiftUI, AppKit, UIKit, watchKit... The number of places that need to run on the main thread is huge, and we aren't even thinking of the individual UI classes within these framework that need main thread synchronization. Every single view or view controller needs to work on the main thread, and thus the need to access the `@MainActor` from everywhere really increases.

To use the main actor, you need to add the `@MainActor` attribute to a definition. It can be either a method or a class. When adding `@MainActor` to a function, the function will always execute on the main thread.

```swift
@MainActor func fetchGames() {

}
```

In the above example, `fetchGames` will always execute on the main actor. This is neat, because this way, future programmers will always know that this code is supposed to run on the main thread, deducing the guesswork and helping you write more obvious code.

If you call a `@MainActor` method outside of the main thread, you need to await on it.

```swift
await fetchGames()
```

Adding the `@MainActor` attribute to a bigger definition such as a class, will make all properties and methods be `MainActor`. Individual methods can choose to not be part of the main actor by adopting the `nonisolated` keyword.

```swift
@MainActor
class GameLibraryViewController: UIViewController {
	//...
	nonisolated var fetchVideogameTypes() -> [VideogameType] { ... }
	//...
}
```

`MainActor` is a really important concept, and learning to use it properly will help you adopt the modern concurrency system easier with any of Apple's provided UI framework. Luckily, its usage is straightforward, and it has no magic or hidden behavior you need to concern yourself with.

# Global Actors

Earlier we said that `MainActor` is a "special" kind of actor. And it kind of is, but it's not the only one of its kind. Turns out that `@MainActor` is a type of actor called a Global Actor.

You know that UI Components are quite literally, all over the place. Different framework have them, and they may be found across files and different imports. To make MainActor work with the UI, there needs to be a way to create an actor that everyone can use when necessary. Global actors, like their name say, are declared globally, and every object interested in adopting them simply need to append it as an attribute, like `@MainActor class MyClassThatRunsOnMainActor`.

Starting on Xcode 13, Beta 3, we can define our own global actors for our own purposes.

**Note:** *The Release Notes for Xcode 13, beta 3, are the the first ones that mention the existence and use of global actors. They weren't mentioned in previous release notes and they were not mentioned in any WWDC2021 session on concurrency. I do not know if it was possible to use them in earlier betas of Xcode 13, but I'm mentioning this because I like small curiosities like this.*

## Creating Global Actors

Creating a Global Actor is as follows:

```swift
@globalActor
struct MediaActor {
  actor ActorType { }

  static let shared: ActorType = ActorType()
}
```

Where `MediaActor` is the name we assigned to it ourselves. Then, every type, method, or even module interested in adopting can do su by appending its name before the declaration, like with `@MainActor`.

Suppose you have a global array that can be written to and read from multiple places at once. That global variable can be attributed with `@MediaActor`, and all operations upon it will run on the same thread, making the actor synchronize the state as necessary.

In the following example, we will create a global `videogames` array, and we will update it from different places.

Start by creating a file called `GlobalState` where we will declare our global actor, global variable, and `Videogame` struct:

```swift
// GlobalState.swift

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

**IMPORTANT NOTE**: *I do not condone the use of global variables this way and there are better ways to abstract it. Remember you are using this project to teach you about global actors, nothing more, nothing less*

Second, we will create a view controller where everything will run in the main actor by default.

```swift
// ViewController.swift

@MainActor
class ViewController: UIViewController {
    
    @MediaActor
    func addRandomVideogames() {
        let zeldaOot = Videogame(name: "The Legend of Zelda: Ocarina of Time", releaseYear: 1998, developer: "Nintendo")
        let xillia = Videogame(name: "Tales of Xillia", releaseYear: 2013, developer: "Bandai Namco")
        let legendOfHeroes = Videogame(name: "The Legend of Heroes: A Tear of Vermilion", releaseYear: 2004, developer: "Nihon Falcom")
        
        videogames += [zeldaOot, xillia, legendOfHeroes]
    }
    
    @MediaActor
    func removeRandomvideogame() {
        if let randomElement = videogames.randomElement() {
            videogames.removeAll { $0.id == randomElement.id }
        }
        
    }
    
    @MediaActor
    func getRandomGame() -> Videogame? {
        return videogames.randomElement()
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        
        Task {
            await addRandomVideogames()
            await removeRandomvideogame()
            if let randomGame = await getRandomGame() {
                print("Random game: \(randomGame.name)")
            }
        }
    }
}
```

I chose this example because the ViewController itself will run on the `@MainActor`, and so will all its properties and methods by default. But, if we are going to interact with the global `videogames` variable, we need to run these methods in `MediaActor`. All three possible operations of the view controller  (`addRandomVideogames()`, `removeRandomvideogame`, and `getRandomGame()`) that need to run on the same actor as `videogames`, can do so, by simply marking them as `MediaActor` .

When we need to access this `@MediaActor` data from the `@MainActor`, the methods are implicitly marked as `async`, so we will need to `await` on them.

So far, `@MainActor` is in many different places. Not only do we have accesses to it in two different files, but also via different declarations. To finish, we will create a file called `Functions.swift` where we will put one function that runs on `@MediaActor`.

```swift
// Functions.swift

@MediaActor
func showAvailableGames() async {
    for game in videogames {
        print("\(game.name)")
    }
}
```

And that's it! You can see how simple it is to implement your own global actors.

# Conclusion

`@MainActor` is a Global Actor. All our UI code runs on the main actor. When we run code that may run on different threads but we need it on the main thread, we can mark the method as `@MainActor` and receive data on it.

Global Actors are useful as they allow us to mark declarations in physical different files, across different declarations, and more. You can create your own global actor if you need to synchronize state across different files and types. Declaring a global actor is easy, and declarations interested in running on them can simply adopt them as `@` attributes.

[Here](/archives/GlobalActors.zip) is the sample project we wrote that makes use of our custom global actor.