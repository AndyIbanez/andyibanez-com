---
title: "Sharing Data Across Tasks with the @TaskLocal property wrapper in the new Swift Concurrency Model"
date: 2021-08-18T07:00:00-04:00
originalDate: 2021-08-16T10:17:17-04:00
publishDate: 2021-08-18T07:00:00-04:00
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
description: "Learn how the @TaskLocal property wrapper can be used to share data across different concurrent tasks."
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
7. [Unstructured Concurrency With Detached Tasks in Swift](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. [Understanding Actors in the New Concurrency Model in Swift](/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)
9. [@MainActor and Global Actors in Swift](/posts/mainactor-and-global-actors-in-swift)
10. **Sharing Data Across Tasks with the @TaskLocal property wrapper in the new Swift Concurrency Model**
11. [Using AsyncSequence in Swift](/posts/using-asyncsequence-in-swift/)
12. [Modern Swift Concurrency Summary, Cheatsheet, and Thanks](/posts/modern-swift-concurrency-summary-cheatsheet-thanks/)

<hr>


Sharing Data Across Tasks with @TaskLocal with the new Swift Concurrency Model

Throughout this tutorial series, we have explored a lot of topics related to concurrency. We have learned the most basic details of how concurrency works, and how we can do more complex work with Detached Tasks.

One particular topic of interest we have mentioned is the Task Tree (refer to the [Structured Concurrency in Swift: Using async let](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/) article for a refresher on the details). The task tree is the result of calling multiple tasks within other tasks - unless they are [detached tasks](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/) - and the tasks in the tree inherit some information from their parent tasks such as priority and context.

Just like tasks can share contextual information, wouldn't be it be great if they could share other data as well? Turns out there is a way to do that: The [@TaskLocal](https://developer.apple.com/documentation/swift/tasklocal) property wrapper. In this article, we will talk about sharing data with this property wrapper across different tasks.

**Note**: *Just like [global actors](https://www.andyibanez.com/posts/mainactor-and-global-actors-in-swift/) (excluding `@MainActor`), the first time I ever saw `@TaskLocal` being referenced was in Xcode 13 Beta 3's release notes. It is not clear to me if the feature was there before and undocumented or if is completely new.*

## Introducing The @TaskLocal Property Wrapper

`TaskLocal` values can be read and written to in the context of a task. The value is shared implicitly and it is accessibly by any child tasks the task create, whether they are `async let` or group tasks.

### Using @TaskLocal

To use this property wrapper, properties marked as `@TaskLocal` must be static. They can be optional or have a default value.

To read their values, you don't need to do anything especial. You can attempt to use the value from anywhere, but if the value was not set by a parent async task beforehand, it will be either nil or the default value you assigned it.


```swift
class ViewController: UIViewController {
    @TaskLocal static var currentVideogame: Videogame?
    // ...
}
```

In the code above, we have created a `TaskLocal` `currentVideogame` property.

If we want to read it:

```swift
// Outside of ViewController
func expensiveVidegameOperation() async {
    if let vg = await ViewController.currentVideogame {
        print("We are processing \(vg.title)")
    }
}
```

Now, if you try to modify `currentVideogame` directly, from anywhere (including `ViewController` itself), you will notice that the compiler won't let us because it's a get-only property.

In order to "assign" it a value, we need to "bind it". To do this, simply access the `TaskLocal`s projected value and you will have access to a binding method called `withValue`.

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

In this sample, we are binding `vg` to our `currentVideogame` task value. All tasks spawned from here on out will have access to it for as long as they are part of the task tree.

Consider the following example:

```swift
class ViewController: UIViewController {
    @TaskLocal static var currentVideogame: Videogame?

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        
        let vg = Videogame(title: "The Legend of Zelda: Ocarina of Time", year: 1998)
        Self.$currentVideogame.withValue(vg) {
            Task {
                await expensiveVidegameOperation()
                Task {
                    await expensiveVidegameOperation()
                    Task.detached {
                        await expensiveVidegameOperation()
                    }
                }
            }
        }
    }
}
```

In the code above, we start launching some tasks after binding our `Videogame`. We start a `Task` where we call `expensiveVideogameOperation`. It will print `We are processing The Legend of Zelda: Ocarina of Time` After it `await`s, we launch another `Task`, which is a child of the current one. Calling `expensiveVideogameOperation` will also print `We are processing The Legend of Zelda: Ocarina of Time`, because this child task has access to the same parent. Things are more interesting when we launch a detached task. When we launch the detached task, we also call `expensiveVideogameOperation`, but this time it prints `No videogame found in the task hierarchy!`. As we discussed when we talk about [detached tasks](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/?utm_campaign=iOS%2BDev%2BWeekly&utm_medium=email&utm_source=iOS%2BDev%2BWeekly%2BIssue%2B519), detached tasks are completely independent and they don't really have a parent to speak of (although they can parents of other tasks, as long as they aren't launched as detached tasks). For this reason, our detached task in the code above doesn't have the `currentVideogame`.

You can freely bind another videogame within the detached task, launch another task, and have access to that value:

```swift
Task.detached {
    await expensiveVidegameOperation()
    let anotherVg = Videogame(title: "Tales of the Abyss", year: 2005)
    await Self.$currentVideogame.withValue(anotherVg) {
        await expensiveVidegameOperation()
    }
}
```

Note the `await` before we bind the value with `currentVideogame`. I am not sure if this is compiler magic, but you will be forced to put an await when you are inside the task. The reasoning makes sense, as `TaskLocal` values can potentially be accessed by multiple threads at the same time, so writing one will prevent our program from having any reace conditions.

# Conclusion

You may find a case in which you need to share a value to all the children in the task hierarchy. When you do, feel free to use the `@TaskLocal` property wrapper. Values will be shared to all the children, but any detached tasks will not have any access to them due to their nature.
