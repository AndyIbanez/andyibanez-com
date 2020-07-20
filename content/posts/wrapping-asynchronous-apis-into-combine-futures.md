---
title: "Wrapping Asynchronous Apis Into Combine Futures"
date: 2020-07-19T22:25:27-04:00
draft: true
---

---
title: "Wrapping Asynchronous APIs into Combine Futures"
date: 2020-07-22T07:00:00-04:00
publishDate: 2020-07-22T07:00:00-04:00
originalDate: 2020-07-19T22:25:27-04:00
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
 - tvos
 - combine
categories:
 - development
description: "Learn to wrap asynchronous APIs into simple Combine publishers in Swift."
keywords:
 - swift
 - ios
 - ipados
 - tvos
 - combine
---

Two of the concepts used a lot in Reactive Programming are the *Future* and *Promises*. Whether you have been using Combine for a while or are new to Reactive Programming, chances are you have seen these two words. These terms date to years ago. And I would be lying if I told you what that they were for until I started learning Combine. The first time I heard about Futures and Promises was back in my NodeJS job half a decade ago, and I didn't understand these concepts back then.

In this short article, we will learn about Futures and Promises and how we can use them in our iOS/iPadOS/watchOS/macOS apps to integrate asynchronous code with callbacks into Combine. Basic knowledge of Combine is assumed for this article.

# But First, a Quick Word on Asynchronous APIs and Combine 

Before we dive right in, there's one thing we need to keep in mind when we work with combine. Combine allows us to grab multiple sources into a single, consistent reactive API. In the case of the iOS SDK, for example, we have a lot of frameworks that may give data either via callbacks (`URLSession`), sometimes app wide notifications (`UINotificationCenter` APIs), and more. Combine allows us to grab all these different sources and *combine* them into one. So dealing with networking callbacks or app wide notifications becomes the same process thanks to the reactive API.

Apple's SDKs provides us with many Publishers for common tasks. For example, where we had `URLSession`'s `dataTask`, we now have a `dataTaskPublisher` as well. And rather than registering callbacks or target-actions for `UINotificationCenter`, we now have a `.publisher` property to work with notifications on Combine/

Despite all this, there are still some APIs that could benefit of a Combine publisher integration but they currently do not.

## Futures and Promises

Promises and Futures are two components of a system that allows us to run code concurrently and have a way to know when such code has finished running. If you google for a definition of futures and promises you are going to find a lot passionated discussions and even some mathematical definitions of these concepts that make it hard to wrap your head around.

You can think of a *future* as a placeholder for a value that doesn't exist yet. This empty placeholder can happily sit forever, idle in your code. When it finally receives a value, in the context of Combine, the Future is essentially a publisher that will send the fulfilled value down the pipeline. The code that fills in this empty value is the *promise*. So, the *future* is the value we want to have, and the *promise* fulfills that value for us. You can imagine this system makes it very easy and intuitive to work with asynchronous code that gives us a value back.

## Combine-fying Non-Combine Asynchronous Code Into a Future Publisher

As I said earlier, there are some elements in Apple's SDKs that could benefit from a Combine publisher, but they don't provide such a thing. One examples I can immediately think of is the `LocalAuthentication` framework, when you need to ask for permission to use Touch ID or Face ID.

In the case of the `LocalAuthentication` framework, you can simply create a *Future* directly:

```swift

public enum BiometryError: Error {
    case localAuthenticationFrameworkError(errorPointer: LAError)
    case evaluationError(error: Error)
    case unauthorized
}

let biometryFuture = Future<Bool, BiometryError> { completion in // 1
    let context = LAContext()
    let biometricPolicy = LAPolicy.deviceOwnerAuthenticationWithBiometrics
    
    var error: NSError?
    
    if context.canEvaluatePolicy(biometricPolicy, error: &error) {
        
        context.evaluatePolicy(biometricPolicy, localizedReason: "To log in") { (success, error) in
            if let error = error {
                completion(.failure(.evaluationError(error: error))) // 2
            } else if success {
                completion(.success(true)) // 3
            } else {
                completion(.failure(.unauthorized))
            }
        }
        
    } else if let error = error as? LAError {
        let errorWrapper = BiometryError.localAuthenticationFrameworkError(errorPointer: error)
        completion(.failure(errorWrapper))
    }
}
```

This looks like quite a mouthful, so let's give a quick rundown of what's going on here. It's worth noting that the handful isn't even the Combine Future/Promise part:

1. We are creating a future that will return a `Bool` on success, and a `BiometryError` on failure.
2. We can use the completion handler. This allows our `Promise` to notify the `Future` that a value is ready. In this case, we provide a `BiometryError`
3. We were able to successfully get a biometric scan, so we pass in the right value.

And that, is a perfectly valid publisher. You can use it as such with the operators you know and love.

```swift
biometryFuture
    .sink(receiveCompletion: { (completion) in
        // Handle any errors here
    }, receiveValue: { (value) in
        print("Did Login Successfully: \(value)")
    })
.store(in: &subscriptions)
```

# Conclusion

We can *Combine* anything into publishers, even if we do not have native publishers for it. Using Futures and Promises is a good way to make simple asynchronous code into Combine code.

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.