---
title: "Nsnotificationcenter"
date: 2019-10-17T14:17:01-04:00
draft: true
publishDate: 2019-23-09T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - macos
 - tvos
 - watchos
 - nsnotificationcenter
categories:
 - development
description: "Learn how to use the powerful NSPredicate API for searching and filtering."
keywords:
 - swift
 - nsnotificationcenter
 - ios
 - tvos
 - ipados
 - watchos
---

Sometimes when you are writing an app, you need to be notified of events occurring somewhere else in the system - either in your own app, or in the operating system - and react to them accordingly. For example, you may be an app like Snapchat and you want to know when a screenshot has been taken. This is a system notification that you can "listen" to in order to react. If you have an app like a photo gallery, you may need to know when the user adds a new photo so you can update all relevant UI and make other necessary updates.

`NSNotificationCenter` allows you to listen to events and to react to them. In this article we will explore how to make use of this API in our apps, we will listen to system-provided notifications, and we will also implement our own.

# Introduction

Not to be confused with the Notification Center feature on Apple's devices, `NSNotificationCenter` allows apps to listen to events and to react to them. Apple describes it as "A notification dispatch mechanism that enables the broadcast of information to registered observers."<sup>[1](https://developer.apple.com/documentation/foundation/nsnotificationcenter)</sup>

Essentially, objects that are interested in learning about a certain event can "listen" for it. More than one object can listen for the same event (which is why events are "broadcasted"). Objects who are listening for these events are called Observers.

While you can create your own `NSNotificationCenter` instances, I have never seen this been done in the while. More often that not, you want to use the singleton `defaultCenter`.

# Observing And Posting Custom Events

## Custom Events

You are implementing your own photo gallery app and you want to know when a new app is added so you can update all the relevant parts of the UI. Imagine the Photos.app, where you have the main "Photos" tab and the "Albums" tab which has a "Recents" album. How can you update these two when a new image is added?

The solution is to use `NSNotificationCenter`, register your own `NSNotification` to the system, and observe this notification from both the "Albums" tab and "Recents" album.

### Registering Your Custom Notifications

To register your own notifications, you need to write an extension for the `NSNotification.Name` class and add all your related notifications to it.

```Swift
extension NSNotification.Name {
  /// User added a new photo to gallery.
  class GalleryApp {
    static let newPhotoAdded = NSNotification.Name(rawValue: "com.andyibanez.galleryApp.newPhotoAdded")
  
    /// User deleted a photo from gallery.
    static let photoDeleted = NSNotification.Name(rawValue: "com.andyibanez.galleryApp.photoDeleted")
  }
}
```

What we are doing here is extend `NSNotificationName`, we are adding a class of static objects, and these objects are our own notifications.

<hr>
**Important Note!**

You technically don't need to extend `NSNotification.Name`, and even less add your own class to it. I just find this way to be better if your app implements many custom notifications. The downside to wrapping your own notifications in a class is you lose some of Swift's inference features when working with notifications.
<hr>

<hr>
**Important Note!**

If you have seen other documentation, tutorials, or articles on `NSNotification.Name(rawValue: "")`, you may have seen they don't use reverse DNS notation for the raw value. I believe it is necessary, because other dependencies on your app may create their own notifications as well, and that will create a conflict when you least expect it.

Suppose you add a dependency that takes photos in your gallery app. It may add new notification with the raw value "`photoAdded`" for its own internal use after taking a photo. If our own `newPhotoAdded` notification used that same identifier, there would be a conflict and components in your app may receive the incorrect notification.

It's not always possible if dependencies add their own notifications, and the system provides its own sets of notifications as well, so I strongly recommend you namespace notifications with reverse DNS notation. After all, you will never need to refer to this `rawValue` directly.
<hr>

### Posting Custom Notifications

Now that you have your own notifications, you can start posting them, and anyone interested in them can listen to them.

`NSNotification` center has three overloaded methods to post notifications:

* `NotificationCenter.default.post(notification:)`: Use this method if you just need to post a notification with no context whatsoever. 
* `NotificationCenter.default.post(name:object:)`: Use this method if you need to post a notification and care about who is posting it. The `object` person is the sender, the object posting the notification.
* `NotificationCenter.default.post(name:object:userInfo:)`:  This method is the most complete one. You specify the notification name, the object (which is again the sender), and an `userInfo` dictionary. You can use this dictionary to provide additional data to the observers. In our example, we can provide the picture that was just added to the app.

Since we want to pass the photo that was just added, we will use the last one.

```swift
  class PhotoManager {
    func addNewPhoto(_ photo: UIImage) {
      // 1. Our notification name
      let newPhotoAdded = Notification.Name.GalleryApp.newPhotoAdded
      
      // 2. Extra info to pass to the interested parties.
      let userInfo: [String: Any] = [
        "image": photo
      ]
      
      // 3. Post the notification
      NotificationCenter.default.post(name: newPhotoAdded,
                                      object: self,
                                      userInfo: userInfo)
    }
  }
```

Our `PhotoManager` class is responsible for posting notifications. He will notify all interested parties when a new photo has been added to the app.

### Receiving Notifications

The final step is to listen for notifications we are interested in. We will see how our example Gallery app may implement it in its "`PhotosViewController`. The very same logic applies to the "Recents" album.

```swift
class PhotosViewController: UIViewController {
  var photos = [UIImage]()
  override func viewDidLoad() {
    super.viewDidLoad()
    registerForNotifications()
  }
  
  func registerForNotifications() {
    let ns = NotificationCenter.default
    let newPhotoAddedNotif = Notification.Name.GalleryApp.newPhotoAdded
    ns.addObserver(forName: newPhotoAddedNotif,
                   object: nil,
                   queue: nil) { (notification) in
                    // A new photo was added!
                    // Retrieve the photo from the notification and update our UI
                    guard let uInfo = notification.userInfo,
                      let image = uInfo["image"] as? UIImage else {
                        return
                    }
                    self.photos += [image]
                    
    }
  }
}
```

And that's it! Now when our object receives the `.newPhotoAdded` notification. It will add the photo to its own `photos` array. The same thing will happen in the hypothetical "Recents" album.

# System Notifications

The system (iOS, watchOS, iPadOS, macOS, TVOS) can post its own notifications when an event takes place.

You don't need to do anything special to listen to them. Just add yourself as an observe like you did before and you will start to receive these notifications.

There's many documentation pages that list some notification names, like the  [Accessibility Notification Names](https://developer.apple.com/documentation/uikit/accessibility/notification_names), and the [Notification.Name](https://developer.apple.com/documentation/foundation/nsnotification/name) provides a *very long* list of system notifications. It wouldn't be fun to post them all here, so I will just mention the ones I find most interesting:

* [NSCalendarDayChanged](https://developer.apple.com/documentation/foundation/nsnotification/name/1408062-nscalendardaychanged): Posted when the system calendar day changes.
* [NSTimeZoneDidChange](https://developer.apple.com/documentation/foundation/nsnotification/name/1387256-nssystemtimezonedidchange): Posted when the system's time zone changes.
* [NSSystemClockDidChange](https://developer.apple.com/documentation/foundation/nsnotification/name/1414255-nssystemclockdidchange): Posted when the system's clock changes.
* [userDidTakeScreenshotNotification](https://developer.apple.com/documentation/uikit/uiapplication/1622966-userdidtakescreenshotnotificatio): Posted when the user takes a screenshot of the device.
* [brightnessDidChangeNotification](https://developer.apple.com/documentation/uikit/uiscreen/1617832-brightnessdidchangenotification): Posted when the device brightness level changes.
* [SKStorefrontCountryCodeDidChange](https://developer.apple.com/documentation/foundation/nsnotification/name/2909077-skstorefrontcountrycodedidchange): Changed when country used for purchases changes.

You can use these and others in the linked documentation to do more interesting stuff with your code. Play around with the notifications and see what you can do.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.