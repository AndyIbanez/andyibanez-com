---
title: "Generating Feedback Haptics with UINotificationFeedbackGenerator"
date: 2020-01-22T07:00:00-04:00
originalDate: 2020-01-18T17:33:52-04:00
draft: false
publishDate: 2020-01-22T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - macos
 - watchos
categories:
 - development
description: "Learn to implement haptic feedback quickly with UINotificationFeedbackGenerator"
keywords:
 - swift
 - cryptokit
 - ios
 - watchos
---

A few weeks ago, we talked about how we could [play custom haptic feedbacks with CHHapticEngine](https://www.andyibanez.com/posts/playing-custom-haptics-on-ios/). We saw how powerful and flexible that class is, letting us create different haptics for any context.

Sometimes though, you want to play simpler haptics to let the user know that something has occurred. The `CHHapticEngine` class can be overkill, and finding the right parameters to have interaction feedback can be very time consuming.

There is a subclass of `UIFeedbackGenerator` that actually exists since way before we got all the power `CHHapticEngine`: `UINotificationFeedbackGenerator` contains pre-made haptics to let users know when an action finished successfully, with an error, or a "warning" in the context of your app.

# Simpler Haptics with UINotificationFeedbackGenerator

In the previous post, we saw how we could configure all the parameters, like the sharpness and intensity. `UINotificationFeedbackGenerator`, introduced all the way back in iOS 10, is much more simpler than that. In fact, the class has one single method, and that single method can only take three parameters.

Start by creating a `UINotificationFeedbackGenerator`:

```swift
let feedbackGenerator = UINotificationFeedbackGenerator()
```

And then you can use it as so, calling the `notificationOccured(notificationType:)` method:

```swift
feedbackGenerator.notificationOccured(.success)
```

And that's it! `notificationOccured(notificationType:)` takes a `UINotificationFeedbackGenerator.FeedbackType` enum, which can take one of the following three values:\

* **error**: When an error occurs, it will play a harsh feedback.
* **success**: When an operation finishes successfully, it plays a light feedbacl.
* **warning**: Feedback in between `error` and `success`.

It's very easy to add this type of feedback, so I recommend you add it whenever it makes sense in your app to have haptics, whether it is something like saving a file successfully, tweeting something successfully, a network error, and so on.

# Conclusion

Haptics are a core part of the iOS experience, and implementing them doesn't have to be complicated. If you need a lot of control over your feedbacks, use `CHHapticEngine` directly. If you need simple feedbacks that respond to user interaction, the `UINotificationFeedbackGenerator` class is likely to cover your needs.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.