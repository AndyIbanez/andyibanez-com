---
title: "Common Reasons for Background Tasks to Fail in iOS"
date: 2020-08-05T07:00:00-04:00
publishDate: 2020-08-05T00:00:00-04:00
originalDate: 2020-08-02T22:31:26-04:00
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
 - backgroundtasks
 - wwdc2020
categories:
 - development
description: "Learn what are the common reasons your background tasks never get executed, and how to go around them."
keywords:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - backgroundtasks
 - wwdc2020
---

[Apple introduced modern background tasks last year on iOS 13](https://www.andyibanez.com/posts/modern-background-tasks-ios13/). These new APIs have been out for a little over year (counting the beta period). Many developers have tried to adopt them to moderate success. Many of them have found them to be very unpredictable and that only work a fraction of the time. If you look around the internet (and even on the comments in that article), you will see many developers weren't able to get them to work as expected.

In the article linked above, we mentioned that you can schedule your tasks, but it is entirely up to the system to decide when they will be executed, and if at all. In this article we will explore some common reasons the system may decide to ignore your tasks, so you can hopefully work around the limitations and setup more predictable background tasks.

In this article, we will talk about 7 factors that impact background runtime. These factor may apply to your background tasks whether they are using `BGAppRefreshTask`, `BGProcessingTask`, background push or URLSession backgrounded tasks.

# It Is All About Balance.

At the end of the day, what matters is balance. Balance between what your app wants to do and what the system needs. In a constrained system where hundreds of apps may be fighting for background resources, the system has to choose who will get to use its battery life and other performance features. After all, the system needs to maintain all-day battery life, be performant at all times, respect users' privacy, and respecting user intent.

# Factors That Affect Background Task Execution.

There are actually more than seven factors that affect background task execution, but many of them don't require any action. Factors such as the device lock state, if the screen is on or off, there is an iCloud restore going on, the camera is in use are just natural states of the device that you can't do anything about.

The factors we care are about will be listed below. These will either require you to do things differently, will require your users to do things differently (a very important factor), or they will require you to understand how resource budgeting work within the system to assign resources. I apologize if some of these are obvious - My source for this article is the "Background execution demystified" WWDC2020 session vide.

## Critically Low Battery

When the battery is at %20 capacity or lower, we can say it is at a critically low battery. The system will choose to preserve battery life by suspending non-essential processes and will prioritize essential work only.

Background tasks are *opportunistic*. If they have a chance to run, they will. You should never rely on background tasks for core functionality of your app. Opportunistic tasks get suspended the moment the device is in critically low battery.

## Low Power Mode

Low power mode is similar to having a critically low battery. The main difference is user's can trigger it at will, but the side-effects of it are the same.

The system will cancel background processing and opportunistic tasks will be suspended. The device will perform essential activity only.

This one is important because a lot of people these days enable low power mode and have it on all the time.

You can use `NSProcessInfo.processInfo.isLowPowerModeEnabled` and `NSProcessInfoPowerStateDidChange` to query whether low power mode is enabled and to learn when it is triggered by the user.

## App Usage

There are times when the system must prioritize some apps over others. Whenever it is necessary, the system will prioritize apps that are most important to the user. If your users are not using your app as much as other apps, then it will simply be treated as a low priority app by the system. The system uses machine learning to predict what apps the user will use and when, and it will use this information to prioritize background activity.

## App Switcher

Sometimes the system may have hints about what apps the user wants to run, even if they are not part of what the system has learned about their habits.

One such hint is the App Switcher. Apps that are currently backgrounded and visible in the App Switcher have higher priority by the system.

This is very important. **The system will constrain the apps that can run background tasks to apps that are visible on the App Switcher**. I have always said that killing apps on iOS is an enemy of the system, because the system is smarter at resource management than humans. Unfortunately, way too many people kill apps in the App Switcher. Those users may never benefit from having the system honor their background task requests.

We all know a person or two who this. If your users ever report to you that your app doesn't seem to execute background tasks, this is probably one of the first questions you should ask them.

## Background App Refresh switch

Apps that support background task execution can be enabled and disabled by your users at will. Using the Background App Refresh screen in the Settings app, users can enable and disable background app refresh for specific apps.

The switch is enabled by default, so users have to know what they are doing in order to turn it off and what that implies in terms of your app's background execution.

You can query the system to know if Background App Refresh is enabled for your app by using `NSProcessInfo.processInfo.backgroundRefreshStatus` and the `UINotification.backgroundRefreshStatusDidChangeNotification` notification to learn when it is toggled.

## System budgets

Because the Background App Refresh switch is enabled by default, many users may not know their apps support background task execution, or even what that means. This is why System budgets are important to understand how your app may execute in the background when you have to compete against hundreds of other apps.

The system has both energy and data budgets. These budgets are slowly distributed to different apps throughout the day. When an app runs, it deducts from these budgets - there is no mistake with what I wrote here. The budgets are decreased when your app *runs*, not only when it manages to execute a background task.

If you are a good player and are not running obnoxious lengths of code, the system will have no reason to limit your app.

## Rate limiting

To help apps with budget management, the system may impose rate limiting on some apps. The system will space out background task launches for you. When you ask the system to schedule a background task for you, the system will decide the optimal time to honor your request.

This is one of the sources of confusion for this new system because developers keep scheduling tasks but don't see any execution on them. Remember, you can ask the system to execute a task, but it's up to it to honor your request and what capacity. It may trigger it later than you expect to, earlier, or never.

# Control for Developers.

Most of the factors seen above are external, either due to user settings or system resources management. But there is one factor you have slight control over to improve your chances of the system honoring your requests in a timely manner.

As we said above, if you are a good player and perform little work per launch, you are being responsible with the budgets assigned to you. But you can do more, and you should prevent your app from using unnecessary resources while it is running.

## Power Consumption

Minimize power consumption. Do not use hardware you have no need for. Do not use GPS and accelerometer, or other hardware that needs to be constantly in unless you really need to. When using the GPS, ensure you configure it to receive less events at the cost of slightly less accuracy if it is acceptable.

Do try to finish work as quickly as possible. If dealing with APIs that require you to call a system completion handler, properly signal the system that you have finished working. This way, the system will know to not only limit your app after your task is done, but it will also know how good of a player you are. For app refresh tasks, always call `setTaskCompleted(success:)` when you are done.

## Data Usage

You should also minimize the amount of data your app uses. When called for background refresh, only download what you need to update your UI and nothing more. If you use images, download thumbnails and not full images.

You should enqueue URLSession background transfers when you need to download data.

**Ideally, keep the amount of data you download to 100 Kilobytes or less while your app refreshes**. This is a good guideline for both app refresh tasks and background pushes,

## Background Pushes

Also known as "silent push notifications", you can send background pushes that doesn't show any indication to the users of their reception. They are instead, notifications for the app itself. They are low priority and preserve power.

They are great to let the app know that new content is available, but it's not immediately necessary for the user. 

### Silent Pushes and the Seven Factors

The seven factors come into play for background pushes, except for App Usage.

The system will not limit background pushes based on App Usage. So no matter how much your user uses your app, whether it is a bit or much, background pushes will not be gated by app usage metrics.

This does not mean background pushes are a free way to trigger your app to do background work. Rate limiting is still a thing when it comes to this mechanism, and you should be responsible with them. If you are sending a lot of silent pushes, you are depleting budgets and affecting system resources.

To avoid this, the system may delay the delivery of some background pushes. If you send 14 pushes in a short time, the system will not wake up your app for each. It may coalesce them into 7 pushes to optimize resource usage.

Rather than focusing on the delivery rate, focus on spacing the number of pushes sent to your apps. Send them less often. Background pushes are for non-essential content such as receiving messages in a muted conversation in a chat app.

## Background URLSessions

When you use Background URLSessions, the system is in charge of managing them. This allows the system to continue the transfers even after the app exists (as long as it has not been killed from the App Switcher). They are not constrained by run time limits.

They are good for file downloads and photo uploads.

You can configure them to not use cellular data and you can configure them to send launch events. This allows the system to notify us when a big file download or upload has completed.

If you use the `.isDiscretionary` property, the work can be deferred until later. Enqueued tasks are always discretionary. The system will choose the right time to execute these tasks, such as when the phone is plugged into a charger and on a Wi-Fi network.

### The Seven Factors and Background URLSessions

Most of the factors apply to background URLSessions depending on the configuration. If the app was brought to the foreground recently, rate limiting won't apply.

For non-discretionary launches that may even be queued, only App Switcher and System budgets apply. If the user kills your app while it is executing one of these tasks, the task will stop. System budgets apply, but they are relaxed and not as strict. This is because the system knows the user wants this to happen to some capacity.

If you configured your task to send launch events, App Usage and  Rate limiting won't apply.

## Background Processing Tasks.

App Usage, App Switcher, Background App Refresh switch, and Rate limiting are the only factors that apply to Background Processing tasks. If the user has used your app at least once in the last few weeks, the system may honor the launch of this task, so the system is very flexible with this type of tasks. If the user charges their phone daily, your tasks are more likely to run.

# Conclusion

The system will ultimately have all the control on whether your tasks will be executed or not. Still, if you are a good player and your apps are not resource hogs, you should have a better change of getting executed, depending on the conditions, task type, and whether your users are actually using your app.

