---
title: "Playing Custom Haptics on iOS"
date: 2019-09-28T16:47:11-04:00
draft: false
publishDate: 2019-10-02T07:00:00-04:00
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
 - corehaptics
 - ios13
categories:
 - development
description: "Learn to use the different NSFormatter subclasses to format data in a human-readable way."
keywords:
 - swift
 - ios
 - watchos
 - apple
 - programming
 - corehaptics
---

Vibration and physical feedback has become an important feature of Apple's operating systems. Starting with the Apple Watch, Apple replaced the traditional vibration feedback with haptic feedback hardware, which allowed them to have more control over how vibrations and physical feedback work.

First being introduced in the Apple Watch, Haptic Feedback has been a core part of the Apple ecosystem experience since. Just think about it, wouldn't it be weird if you force-touched the screen to do something, and there was no physical response from the device? Think of the old Peek-and-Pop. If you updated to iOS 13 and got context menus, you may feel they feel great to do in iOS (because they have a haptic engine), but on iPad they feel lacking because the device doesn't vibrate when triggering them.

Haptic feedback is present even in the most unexpected places of Apple's lineup. Their touch-pads respond with haptics. The iPhone 7 and 8, which have no physical home button, create exactly the same feeling by providing a haptic instead of physical touch presses. We may not notice it, but if you are deep into Apple's ecosystem, haptics are all over the place.

In this article, we will explore how we can integrate haptic feedback into our iOS 13 apps.

# Playing with Haptic Feedback on iOS 13.

Apple has a few APIs that give developers a lot of control over the haptics they want to create on iOS 13.

The [CHHapticEngine](https://developer.apple.com/documentation/corehaptics/chhapticengine) handles requests to to play haptic patterns.

To use it, you need to `import CoreHaptics`.

## CoreHaptics Setup

Before you actually try to play some feedbacks, though you need to ensure that the device supports them. As we discussed earlier, not all devices support them, and a notable example of that are all iPads released until now.

To check if your device supports haptics, the `CHHapticEngine` object has a static method called `capabilitiesForHardware()` that returns the supported feedback capabilities.

```swift
guard CHHapticEngine.capabilitiesForHardware().supportsHaptics else {
	// Haptics not supported.
	return
}
```

Once you know your device supports haptics, you can create one like this:

```swift
var engine: CHHapticEngine?
// ...
do
{
  engine = try CHHapticEngine()
  try engine?.start()
} catch {
  print("Problem with haptics: \(error)")
}
```

It's not clear to me at this when the engine can throw an error as there's a tremendous lack of documentation at this time, even after iOS 13 has been released. My assumption is when you try using it in a device that doesn't have a haptic engine, or when the hardware inside the device is malfunctioning.

The feedback engine is the connection to the haptic server inside a device. You can create multiple connections to this server (via multiple instances of `CHHapticEngine`), so don't worry about treating it as a singleton. Each connection is independent of the others.

The call to `start()` is asynchronous, so be wary of that. There is an asynchronous variation that takes a completion clock if you need it:

```swift
  engine?.start(completionHandler: { (error) in
    // .. Handle error if relevant
    // .. Do something after it starts
  })
```

There's some additional properties you can set. If the Haptic Engine gets stopped for any reason. When there's an error with the haptic server, you can also get notified when it has been reset.

Bellow are the two relevant properties:

```swift
engine?.stoppedHandler = { reason in
  print("Haptic engine stopped due to reason: \(reason)")
}
          
engine?.resetHandler = {
    print("Haptic server has been reset")
		// You can try to start it again.
    do {
      try engine?.start()
    } catch {
      // handle error.
    }
}
```

## Playing Haptic Events

After you have done all that setup, you can start creating some haptics events. You create an event, request a player from the server, and send your haptic requests to that player.

To create a haptic event, you begin creating the parameters that will be used to configure it.

```swift
let intensity = CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5)
let sharpness = CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.5)

let event = CHHapticEvent(eventType: .hapticTransient, parameters: [intensity, sharpness], relativeTime: 0)

do {
    let pattern = try CHHapticPattern(events: [event], parameters: [])
    let player = try engine?.makePlayer(with: pattern)
    try player?.start(atTime: 0)
} catch {
    print("Failed perform haptic: \(error.localizedDescription).")
}
```

These are just some of the basic configurations you can do for your feedbacks. Alongside vibrations, you can play sound as part of an event so you can create feedback similar to that of the context menus in Springboard. You can even read and write haptics into a file so users can create their own. In the Settings app on iPhone, you can create custom vibration patterns for calls, and they all use haptic feedbacks.

# Conclusion

Using Haptics on Apple's devices is now easier than ever and we have a lot of flexibility to implement them. When designing apps, consider the interactions where you could use haptics to improve the experience of your users, but be careful not to overdo them because if used wrongly, they can cause physical discomfort. Try to see where devices that support use it so you can learn the best place to implement it.


<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.

