---
title: "Modern Concurrency in Swift: Introduction"
date: 2021-06-16T07:00:00-04:00
originalDate: 2021-06-13T00:17:10-04:00
publishDate: 2021-06-16T07:00:00-04:00
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
description: "Learn about the modern concurrency features intorduced in Swift 5.5, at Apple's WWDC2021."
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

# Introduction

This is a tutorial series focused on the new async/await APIs Apple introduced in WWDC2021. I do not know how many articles it is going to have yet, but they will be posted in the upcoming weeks.

The WWDC2021 session videos do a great job explaining these new APIs, but I feel they can still be overwhelming for both newcomers and long-time developers alike. My intention with this series is to talk about the new concurrency APIs, one step a time, covering a few concepts on each article, until you can feel confident with your understanding of these APIs. When I see it necessary, I may use or modify Apple's provided snippets. I will explicitly mark this external code as such.

The knowledge you will see throughout the series is knowledge I have obtained from the WWDC2021 sessions (the relevant sessions will be linked in each article), playing around with them myself, and other sources. I do not claim to know everything about the new await/async APIs, and while the Evolution proposal was approved ahead of WWDC2021, I am building this series with the knowledge I have as I did not explore the proposal before WWDC2021

For that reason, please do point out inaccuracies if you find them so I can fix them. It is very important for me that this series is as clear and accurate as it can be. Please keep an eye out on typos and report anything odd you find, either via E-mail or Twitter.

Before we explore the new APIs, let's talk about the current concurrency implementations and their problems. By the end of this introductory article, you will be convinced that the new APIs are worth investing your time in.

# Concurrency and its Problems with Current Implementations

In WWDC2021, Apple introduced a new way for developers to implement concurrency in their apps. I will refer to them as the "async/await APIs", as these two words are at the core of it all.

As developers, we have used concurrency, oftentimes without knowing. Almost every call that takes a closure in the iOS SDK has such signature because it is an asynchronous call. If you have been an iOS developer for a while, you know that UI code runs in the so-called Main Thread. Because UI manipulation takes place here, if anything takes a very long time to finish, the system may decide your app has frozen, and it will kill it, but not before your users realize your app is in a hanged state. The need for concurrency in software in general will, more often than not, be about spawning *concurrent* tasks to get some job done, or to speed up something. When it comes to Apple technologies, the need for concurrency is the same, but we also need to keep an eye out for the main thread to not get blocked by anything.

Calls that have the potential to freeze our main thread are all over Apple's SDKs. This is why Apple provides us with different tools to delegate work to different threads and keep our main thread free.

Before moving on, remember that these new APIs are guaranteed to become the standard, but they are not the only ones used for concurrency. I have a [full article on the alternatives](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/) if you find the async/await APIs don't cover your needs.

## Callback-based concurrency for API for consumers

Take the `URLSession` API as an example. Prior to WWDC2021, if you needed to some sort of networking call, you would call something like this:

```swift
// ... (1)

let task = URLSession.shared.dataTask(with: ...) { data, response, error in
    // ... (2)
}.resume()

task.resume()

// ... (3)
```

Anything that goes inside the callback closure - that is, everything within the braces `{}`, is code that will be called *asynchronously* after the download has taken place and there's no guarantee on what order it will be called in. We know it will be called after `(1)`, but that's about it.

In the snippet above, we have code that executes before the network call `(1)`. But `(2)` will not be executed immediately. Instead, execution *may* continue for `(3)`, and when the download has finished `(2)` will be executed. The execution order for `(2)` and `(3)` is not guaranteed. In this particular example, we can say that "obviously" a network call is slower than the linear execution of a program, but don't take this for granted - there's plenty of APIs that don't hit the network that are asynchronous in nature.

This works, and this old-style API is not going anywhere. But what if we need to later parse some JSON, or do more network calls based on the response? This becomes painful and we arrive to what we call a *pyramid of doom*.

```swift
let task = URLSession.shared.dataTask(with: ...) { data, response, error in
    let taskThatNeedsPreviousResponse = URLSession.shared.dataTask(with: ...) { data, response, error in
        let evenMoreNestedNetworking = URLSession.shared.dataTask(with: ...) { data, response, error in
            /// We can finally do more work here
        }
        evenMoreNestedNetworking.resume()
    }
    taskThatNeedsPreviousResponse.resume()
    
}.resume()

task.resume()
```

As the calls get nested and nested (and nested), it can become a problem when it comes to readability. You can take action and move every "pyramid floor" into its own function, but that's more of a patch than a real solution as you end up polluting your scope.

## Callback-based concurrency for API Designers

Now suppose you were tasked with creating a function that downloads an image and resizes it to create a thumbnail. You may end up writing something like this:

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(nil, error)
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(nil, FetchError.badID)
        } else {
            guard let image = UIImage(data: data!) else {
                return // (1)
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
                guard let thumbnail = thumbnail else {
                    return // (2)
                }
                completion(thumbnail, nil)
            }
        }
    }
    task.resume()
}
```
*(This code was taken directly as-is from Apple's [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/) session)*

The first thing you will notice is that this code is a mouthful. Just sit down and appreciate how *long* it is. It begins by downloading an image, and then it tries to resize it. Both the network call and the thumbnail resizing call are asynchronous calls. Not only that, but your call needs tot pass in their own completion handler as well.

This code has bugs too, and they may be hard to find. Remember that if you write a callback-based function, you need to call the passed callback regardless of what happens. The developer calling your function can find cases in which their callback is never called. In the example above, there's two places where this can happen. I have marked those places as `(1)` and `(2)`. As you are not calling the completion handler in these places, you will leave the API caller waiting for a response that will never arrive - at the very least you won't be blocking the thread, though.

So, the first problem we can find here is that you are responsible from calling the callback when you are done with your job. This isn't too bad for small functions, but it can become overwhelming when you realize there's many edge cases you need to think of.

But one thing I have always disliked about callback-based APIs is that all the information about the "return" type and the error are part of the closure you are given. Because of this, you cannot have a clean API that states its return type and whether it can yield an error not. There's no such thing as throwing an error. With these APIs. You have to provide them to the callback. While static typing does not disappear, it does get more abstracted (not to mention autocomplete isn't as useful, as there times it decides not to work when making such calls). And as a cherry on top, your API consumers may decide to discard the error if they so choose - this is not necessarily a bad thing, but there's times when you want them to *really* take some action.

# Combine?

The Combine framework solves many of the problems above beautifully through the use of pipelines, but we will not be talking much about Apple's reactive framework in this series, and there's a few reasons for that.

First, I just feel Combine's future is uncertain. I love the framework myself, and while at the beginning of WWDC2021 [I was skeptic about whether these new APIs could take its place](https://twitter.com/AndyIbanezK/status/1402463420931317762?s=20), I changed my opinion after I watched some more sessions on the topic.

Second, I feel it's not used enough. Combine was introduced in 2019 and it has(had?) a big role driving the existence of SwiftUI. But I just get the feeling that it hasn't seen much adoption in the few years it has been out in the wild. There's no evidence people are adopting it to replace their callback-based code, and both the lack of community resources (a few exist, and are awesome) and lack of updates on the framework make it seem that it may not be wise to invest much time on it until we know what Apple's plans for it are.

Combine will not be mentioned much throughout this tutorial series unless it's relevant. In general, I no longer consider it a candidate to replace callback-based code - I was a huge fan of wrapping asynchronous code in Futures, though.

# A new way of thinking

Finally, before diving in to the articles below, I recommend you try to throw your current knowledge of concurrency outside the window, because the implementation for async/await is very different, and it's important to understand this mindset before you truly understand how it works. Once you understand async/await, the rest of the toolset is easier to understand.

I am not saying your current concurrency knowledge will be irrelevant. Far from it, but it's interesting how imposing an easier to write concurrency code requires us to rethink how we have been thinking about concurrency in Apple's platforms in the past decades. I actually think async/await is easier to understand for people who have never seen asynchronous code before, because the way it's implemented doesn't derive much from procedural programming.

# Without further ado...

The table of contents below list the articles of this series. As the series is not complete at the time of this writing, this table may change overtime, adding or removing topics, or reordering them, as the series progresses. They are designed to be independent of each other, so you don't need to read the early articles if you just need the last ones. That said if you are new to async/await, you should read them all in order.

The tutorials contain code that you can run. Feel free to copy and paste it or download the sample projects when available to aid your learning.

# Table of Contents

1. [Understanding async/await in Swift](/posts/understanding-async-await-in-swift/)
2. [Converting closure-based code into async/await in Swift](/posts/converting-closure-based-code-into-async-await-in-swift/)
3. [Structured Concurrency in Swift: Using async let](/posts/structured-concurrency-in-swift-using-async-let/)
4. Unstructured concurrency
5. Understanding Actors in Swift
6. The AsyncSequence Protocol in Swift