---
title: "Benchmarking Your App with MetricKit"
date: 2020-10-07T07:00:00-04:00
draft: false
originalDate: 2020-10-07T09:51:03-04:00
publishDate: 2020-10-07T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - wwdc19
 - wwdc20
 - metrickit
 - swift
 - programming
 - apple
 - ios
 - ipados
categories:
 - development
description: "Learn how to use MetricKit to understand the performance of your app."
keywords:
 - wwdc19
 - wwdc20
 - metrickit
 - swift
 - programming
 - apple
 - ios
 - ipados
---

Sometimes we may be interested on how well our app is performing out there in the world. After all, our apps may be running in different environments that are hard to test or that Instruments may not catch.

For this purpose, Apple introduced MetricKit back in WWDC2019. MetricKit allows us to aggregate and analyze this benchmark data on a per-device basis, and not only does it include information on performance and battery usage, but also on exceptions and crash reports.

MetricKit will provide us with data of the last 24 hours at most, an it will include all kinds of metrics.

# Implementing MetricKit

In order to implement and start using MetricKit, we need a place in our app that never gets destroyed and is always there, receiving system events. A good candidate for this is our AppDelegate. All we need to do is to register ourselves to the `MXMetricManager` object, extend our AppDelegate to conform to `MXMetricManagerSubscriber`, and implement the `didReceive(_:)` method of this protocol.

```swift
// In AppDelegate
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
    MXMetricManager.shared.add(self)
    return true
}

// ...
// Creating an extension to conform to MXMetricManagerSubscriber
extension AppDelegate: MXMetricManagerSubscriber {
    func didReceive(_ payloads: [MXMetricPayload]) {
        // Handle payloads here
    }
}
```

Around every 24 hours, MetricKit will call the `didReceive(_:)` method and it will handle us many *payloads*. A Payload is simply an object that wraps certain metric data. If you explore the properties of this object, you will find the `animationMetrics` property, which is a `MXAnimationMetrics?` - like you can tell, it contains metrics related to animations. You will also find `applicationLaunchMetrics` (`MXAppLaunchMetrics?`), `cellularConditionMetrics` (`MXCellularConditionMetrics`), and many more.

It's worth noting that a `MXMetricPayload` contains a bunch of metrics, but it may not contain all the metrics in a single payload. That is to say, you may receive a payload that has the `applicationLaunchMetrics` property filled in, but not the `animationMetrics` one, and so on. This is why it is important to actually iterate through the array and check what metrics are filled in.

How can we test this when we expect metrics to be called every 24 hours? Luckily, Xcode has a way to simulate metrics. Do note that this doesn't work on the simulator and you will need to attach your physical device.

To emulate metrics, go to the `Debug` menu in Xcode, and then click `Simulate MetricKit Payloads`.

## Manually Logging Critical Sections of your Code

By default, MetricKit will give us a lot of information regarding the usage of our app, but you can also manually log critical sections of your app. You can use this to benchmark code that you suspect is taking too long, for example, and understand better the behavior of your app under certain specific scenarios.

```swift
let filterLog = MXMetricManager.makeLogHandle(category: "Picture Filter")

func applyFilter(nanmed name: String) {
    mxSignpost(.begin, log: filterLog, name: "\(name) filter")
	// Long running operation of applying a filter here
	// ..
	// Don't forget to call end to end the data collection. This can go inside a completion handler as well.
    mxSignpost(.end, log: filterLog, name: "\(name) filter")
}
```

## Sending Metric Data to a Web Service.

You may want to process and store the metric information to help you understand your users' habits and your app performance in the long run.

Apple thought it through, and all metrics that inherit from `MXMetric` have a `jsonRepresentation` property you can use to easily send this over to a custom web service.

## WWDC2020 Improvements

The basics of this article covers how you can use MetricKit. Some metrics may be unavailable on iOS 13. This section will cover what's new with this framework in iOS 14 so you don't expect it to work in iOS 13.

Some new tasks we can do now include:

- CPU Instructions
- Scroll Hitches, for when scrolling through a table view or other scrollable components feels laggy. This is good to measure the graphical performance in your app and find reasons for dropped frames or other unwanted visual artifacts.
- App exit reasons

### MetricKit Diagonistics

With Diagnostics, we can get more detailed info for hangs, crashes, disk writes, and CPU exceptions. It allow us to literally get a stack trace of the call site.

To make use of this, all we need to do is implement a new method of `MXMetricManagerSubscriber`. `didReceive(_:)`, but this time, it will pass you an array of `MXDiagnosticPayload` instead of `MXMetricPayload`. And that's it. It will work exactly the same way as the other protocol method. 

The API for dealing with `MXDiagnosticPayload` is pretty much the same as `MXMetricPayload`. Where we had `MXMetric` as the base class for all metric objects, we now have `MXDiagnostic` which fulfills that paper for diagnostics. `MXDiagnosticPayload` will wrap a bunch of `MXDiagnostic` subclasses the same way `MXMetricDiagonistic` does for wrapping `MXMetric` subclasses.

It also includes a `MXCallStackTree` object. This encapsulates stack traces for the moments when regressions occurred. It is important to note that these stack traces are unsymbolicated, so make sure you don't store them in your app unless you want to leak implementation details of certain functionality.

The new subclasses introduced in WWDC2020 for diagnostics are:

- `MXHangDiagnostic`
- `MXCPUExceptionDiagnostic`
- `MXDiskWriteExceptionDiagnostic`
- `MXCrashDiagnostic`

# Conclusion

MetricKit allows us to benchmark our apps. It aggregates a lot of data and hands it to us in reasonable intervals. WWDC2020 introduced the ability to not only do benchmarking but also diagnostics for apps crashing, hanging, and more. Implementing MetricKit in your app is very easy and it's all about implementing two delegate calls (or just one, depending on whether you want diagnostics or not).

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.