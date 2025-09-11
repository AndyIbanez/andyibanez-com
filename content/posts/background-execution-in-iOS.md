---
title: "Background Execution on iOS"
date: 2019-12-18T07:00:00-04:00
originalDate: 2019-12-09T15:51:12-04:00
publishDate: 2019-12-18T07:00:00-04:00
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
categories:
 - development
description: "Learn how to perform background tasks in your iOS App."
keywords:
 - swift
 - ios
 - ipados
 - iOS13
---

In the early days of iOS development, developers had no way at all to perform background tasks beyond a very limited constraints of tasks, like music playback. Modern demands go beyond allowing people to play music in your app while they use a different app, and we as developers need to adapt to these changes. VoIP, lengthy networking, and even silent pushes to keep an app updated are very common these days, and fulfilling these demands used to be hard, if not outright impossible.

iOS 7 was the first one to introduce more slightly powerful background execution APIs. In this article, we will explore how to perform background tasks in our apps and how to give our apps additional time to complete a task that was initially started in the foreground, and was later moved to the background before it had time to complete.

In particular, we will explore the following use cases.

1. Getting additional background execution time when an app is sent to the background.
2. Starting background tasks with silent push notifications.
3. Deferred downloads with Discretionary Background URL Session.

This article is based on the [Advances in Background Execution](https://developer.apple.com/videos/play/wwdc2019/707) WWDC 2019 talk and in the documentation, but I have taken the liberty to add my own code examples and I have removed content that is not not relevant for the majority of developers (VoIP pushes, etc).

<hr>
**Note**

The original version of this article was supposed to cover the old background tasks APIs introduced since iOS 7 and then the new APIs introduced in iOS 13. The article became much longer than I had originally expected, so I separated the article into two different ones: One article for old but still relevant APIs (this one), and another one for the iOS 13 exclusive APIs (the `BackgroundTasks` framework).

For that reason this article won't cover the new shiny `BackgroundTasks`. You will have to wait until December 25 (Merry Christmas!) for that one. Sorry about that!
<hr>

# The Need for Background Execution

## What exactly is background execution?

Background execution is simply letting the app run code while it's not in the foreground - it's not the app the user is currently using.

The app can require background execution if it requests it to the system. If you want to refresh content and have an up-to-date UI for your users when they relaunch your app after a few hours. Chat applications may want to do this, so next time the user launches an app, they can see all their chats updated instead of waiting for the app to do it when they visit the app. The app can also request this to complete some work that started while the app was in the background, like a big network download.

The app can also begin background execution when an event takes place in the system. For example, it can be triggered when the user receives a notification, or when the GPS detects the device is in a specific location. In other words, the data needs to respond to some event.

## Considerations

The background execution system still has some considerations that you should really keep in mind before you go forward with this.

First is *power*. If your app uses too much battery performing background execution, it may be a bad experience for your user. You should let the system know when your tasks finish to become a good player in the Background Execution world. If you allocate 60 seconds for a task and finish it in 40, let the system know and it will become more generous with your requests over time.

The second is *performance*. Multiple apps may be running in the background at a time, and then there's the little bonus that a foreground may also be executing. Be aware that your app is not the only app in your user's devices, and they may have thousands of different apps performing background tasks.

Finally, there's *privacy*. You cannot view other background tasks, as it is expected in the Apple fashion.

You should adopt background execution keeping all these in factors in mind.

# Background Execution Use Cases

<hr>
**Note**

Like I said above, the original article was supposed to cover both the old and new APIs for background execution. Everything written here is stuff that can be done pre-iOS 13.
<hr>

We can divide the use cases of background execution into different categories. If your app needs to do any of the following, you can adopt this new framework:

* Give additional time to execute a task before the app is suspended. If you start work in the foreground, you can complete it in the background.
* Triggering background tasks with silent push notifications.
* Downloading content at a later time with Background URL Session.

We will go through each individually, offering code whenever possible.

## Give additional time to execute a task before being suspended.

`UIApplication` has a simple method that can be used for this: [beginBackgroundTask(expirationHandler:)](https://developer.apple.com/documentation/uikit/uiapplication/1623031-beginbackgroundtask)

```
UIApplication.beginBackgroundTask(expirationHandler:)
```

You should call this method when leaving a task unfinished may cause a bad user experience in your app. You can use it to complete disk writes, finish user-initiated requests, network calls, and tasks similar to that. The `expirationHandler` is optional, but if you provide it the system will call it before the time expires to give you a chance to end a task gracefully before it had time to complete.

You should call this method right before you start your task, and better if you do before the app actually enters the background state.

Each call to this API should have a matching call to [`UIApplication.endBackgroundTask(identifier:)`](https://developer.apple.com/documentation/uikit/uiapplication/1622970-endbackgroundtask). Because apps cannot run indefinitely in the background, you can check how much time your app has by checking the `backgroundTimeRemaining` property of `UIApplication`.

You can call `beginBackgroundTask` any time, and as many times as you want. Each call will return a unique `identifier` you can use to identify a task, or `"invalid"` if the app doesn't support background execution. Don't forget to end your tasks by calling `endBackgroundTask` with the identifier returned by `beginBackgroundTasks`.

You can get your hands dirty with this. Create a new empty project and put this code inside `viewDidAppear` in the default view controller:

```swift
    let taskId = UIApplication.shared.beginBackgroundTask {
      print("We are about to kill your task")
    }
    
    print("The task ID: \(taskId)")
    
    let _ = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
      print("Executing (\(UIApplication.shared.backgroundTimeRemaining) seconds remaining)")
    }
```

In my tests, while this was running in the foreground, it kept printing `Executing (1.7976931348623157e+308 seconds remaining)`. As soon as I backgrounded it, I saw that the system allocated approximately 30 seconds for the background task:

```text
Executing (29.899623215998872 seconds remaining)
Executing (28.80678400799661 seconds remaining)
Executing (27.900681052997243 seconds remaining)
Executing (26.9005714599989 seconds remaining)
Executing (25.804284562997054 seconds remaining)
Executing (24.80326834999869 seconds remaining)
Executing (23.802226524996513 seconds remaining)
Executing (22.801737096997385 seconds remaining)
Executing (21.900720710000314 seconds remaining)
Executing (20.900698297002236 seconds remaining)
Executing (19.900725225001224 seconds remaining)
Executing (18.900687380999443 seconds remaining)
Executing (17.90067362899572 seconds remaining)
Executing (16.900717349999468 seconds remaining)
Executing (15.853402848995756 seconds remaining)
Executing (14.8417572249964 seconds remaining)
Executing (13.82555943299667 seconds remaining)
Executing (12.816451088998292 seconds remaining)
Executing (11.900728770000569 seconds remaining)
Executing (10.900756258997717 seconds remaining)
Executing (9.816338092998194 seconds remaining)
Executing (8.814374344001408 seconds remaining)
Executing (7.805837875996076 seconds remaining)
Executing (6.803114914997423 seconds remaining)
Executing (5.816280047998589 seconds remaining)
We are about to kill your task
Executing (4.802873692999128 seconds remaining)
Executing (3.8997083849972114 seconds remaining)
Executing (2.899573540998972 seconds remaining)
Executing (1.8968817720015068 seconds remaining)
Executing (0.8995601319984416 seconds remaining)
Executing (0.0 seconds remaining)
```
In my tests, the expiration handler got called about 5 seconds before the time expired.

It's possible the system adjusts this automatically depending on how good of a player you are and how you use background tasks.

<hr>

**Important Note!**

While my tests say you get around 30 seconds for execution task and the completion handler gets called about 5 seconds before the task expires, you should not rely on these numbers in your app. Don't hard code anything. As a rule of thumb, just try to finish all your work as soon as possible when using `beginBackgroundTask`, because if the system doesn't adjust to your use case, future iOS versions may change the amount of time you get for executing tasks.

<hr>

Of course we want to be good background execution citizens, so we should end the task at some point. For now, we will end the task when we only have 10 or less seconds left.

```swift
    let _ = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
      let bgTimeLeft = UIApplication.shared.backgroundTimeRemaining
      print("Executing (\(bgTimeLeft) seconds remaining)")
      if bgTimeLeft <= 10 {
        UIApplication.shared.endBackgroundTask(taskId)
      }
    }
```

If you background the app, the app will not print anything else after the available execution time is less than 10. In other words, your expiration handler will not get called when you end the task properly with a few seconds to spare.

For reference, here's the whole code I wrote for `viewDidAppear`:

```swift
  override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(true)
    
    let taskId = UIApplication.shared.beginBackgroundTask {
      print("We are about to kill your task")
    }
    
    print("The task ID: \(taskId)")
    
    let _ = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
      let bgTimeLeft = UIApplication.shared.backgroundTimeRemaining
      print("Executing (\(bgTimeLeft) seconds remaining)")
      if bgTimeLeft <= 10 {
        UIApplication.shared.endBackgroundTask(taskId)
      }
    }
  }
```

### Giving additional time to extensions.

You can also give extensions additional time before they get suspended by the system, with the following method:

```
ProcessInfo().performExpiringActivity(withReason:using:)
```

The first parameter is a string you can use for debugging purposes. The second parameter is a handler where you put code you want to execute in the background. The block will give you a boolean telling you if the process is about to be suspended. If the boolean is `true`, you should take caution to end the task as soon as possible. I wasn't able to find a way to tell how much time you have left to execute a background task with this API.

The system will define if it can execute your handler at all. If it can't, it will call it passing it `true`, forcing you finish everything as soon as possible. If it can execute your task, it will call the handler with `false`.

If the system is executing the handler and needs to suspend it, it will call your handler a second time passing `true`, so keep in mind that this handler might be called more than once. A rule of thumb is to simply cancel anything your handler is doing when the parameter is `true`.

## Triggering Background Execution with Notifications

Background Pushes are a mechanism to tell devices that new data is available without notifying the user. In other words, they don't display any kind of UI or play a sound.

To send a silent push, add the `content-available` key to `1`, and don't include `badge`, `sound`, or `alert`. You must also set `apns-priority` to `5` and it is highly recommended, but necessary on to watchOS,  to set `apns-push-type` to `background`, 

The push will not trigger the download immediately. Instead, the system will intelligently decide the best time to download new content, including factors such as power and performance.

## Download Content at a Later Time with Discretionary Background URL Session.

This is a way to tell the system to defer downloads until a better time. We can provide information to the system for smarter scheduling.

Using it is as easy as any other `URLSession`, but starting on iOS 13, you can set the `isDiscretionary` property to `true`.

```swift
let config = URLSessionConfiguration.background(withIdentifier: "com.andyibanez.pastContent")
config.isDiscretionary = true
let session = URLSession(configuration: config)
```

The advantage of this API is that you have finer control over the session, as you can set intervals, the earliest begin date, and more.

```swift
config.timeoutIntervalForResource = 24 * 60 * 60   config.timeoutIntervalForRequest = 60
```

```swift
var request = URLRequest(url: URL(string: "google.com")!)
request.addValue("foo", forHTTPHeaderField: "bar")
    
let task = session.downloadTask(with: request)
task.earliestBeginDate = Date(timeIntervalSinceNow: 2 * 60 * 60)
```

# Conclusion

Background execution is a very common task and you can adopt your app to do it. Prior to iOS 13, you could follow three use cases to implement it in your own code:

1. Give extra time to a task to finish after it enters the background.
2. Start background tasks upon receiving silent push notifications.
3. Defer long downloads to a discretionary background URL session.

The APIs are easy to use, and I encourage you to add them if it makes sense in your use case. In particular, the first point is very useful as you never know when a user may send your app to the background when your app is doing something important.

