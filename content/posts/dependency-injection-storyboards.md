---
title: "Dependency Injection with Storyboards on Apple Platforms"
date: 2020-05-13T07:00:00-04:00
originalDate: 2020-05-10T18:19:35-04:00
publishDate: 2020-05-13T07:00:00-04:00
draft: false
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
 - tvos
 - storyboard
description: "Learn how iOS 13 solves the issue of Storyboard dependency injection."
keywords:
 - swift
 - programming
 - apple
 - ios
 - macos
 - watchos
 - tvos
 - storyboard
---

Every iOS developer has written a line of code like this one at least once:

```swift
class DollInfoViewController: UIViewController {
  
  var dollModel: Doll?

  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
  }


}
```

Or in the worst case, you may have seen code like this:

```swift
var dollModel: Doll!
```

Then, when you want to create a view controller of that type you'd do:

```swift
// First we need to check if we can actually instantiate the view controller.
guard let dollInfoVc = storyboard?.instantiateViewController(withIdentifier: "DollInfo") as? DollInfoViewController else {
    fatalError("Unable to load view controller.")
}

// Then we pass in the data we want to work with. 
dollInfoVc.dollModel = doll
```

Code like this is *very* error prone. The worst part is that up untul iOS 13, it was pretty much necessary to pass data around from view controller to view controller. It's not possible to do this in any different way in different iOS versions.

iOS 13 solves this very elegantly, by introducing Dependency Injection on Storyboards.

# A Quick Introduction to Dependency Injection

Dependency injection is nothing more than to pass the data to the object that need them in order to work. Generally, you do this as part of an object's initializer. When working with Storyboards, it was never possible to pass the data to the view controllers that need them directly. You always needed to specify a property for the data and then fill in that data later. This is why Storyboards are usually plagued with (forced unwrapped) optionals.

iOS 13 intoruces new APIs to make dependency injection possible, without having to fill the destination view controller with optionals.

# Dependency injection with Storyboards.

iOS 13 introduced the [instantiateViewController(identifier:creator:)](https://developer.apple.com/documentation/uikit/uistoryboard/3213989-instantiateviewcontroller) method to `UIStoryboard`. This method takes a `identifier`, which is the identifier of the storyboard itself which you know and love, and a creator block. The creator block includes your custom initialization code for your storyboard. The creator block gives you a `NSCoder` object which you need to complete the initialization of your view controller.

To use this method, we need to do a few things.

First, you can finally create an initializer in your view controller that takes in the data you require. This initializer must take a `NSCoder`. Because of this, our `dollInfo` property also loses the question mark. Neat!

```swift
class DollInfoViewController: UIViewController {
  
  var dollModel: Doll

  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
  }

  init(coder: NSCoder, doll: Doll) {
    self.dollModel = doll
    super.init(coder: coder)
  }
}
```

Second, you should make the default `init(coder:)` initializer to fail. You need to do this, because this view controller now REQUIRES a `dollModel`, which is what you want.

```swift
init(coder: NSCoder) {
  fatalError("You must provide a Doll object to this view controller")
}
```

The rest of the work takes place in the view controller that want to call you.

```swift
guard let vc = storyboard?.instantiateViewController(identifier: "DollInfo", creator: { coder in
    return DollInfoViewController(coder: coder, doll: doll)
}) else {
    fatalError("Failed to initialize view controller")
}
```

With this, we can finally initialize view controllers using dependency injection! You now longer need to specify public properties for your objects, and better yet, you no longer have to deal with optionals that can be error prone.

# Conclusion

Dependency injection is nothing new. Unfortunately, using it with Storyboards has never been directly possible. Thanks to iOS 13 we can now do this. Eliminating optionals has never felt better!


