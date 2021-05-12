---
title: "Integrating FaceID/TouchID with SwiftUI"
date: 2020-05-12T07:00:00-04:00
originalDate: 2021-05-09T22:09:16-04:00
publishDate: 2020-05-12T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - faceid
 - touchid
 - swiftui
 - swift
 - programming
 - apple
 - ios
 - ipados
categories:
 - development
description: "Learn the right way to implement FaceID or TouchID with your iOS app using SwiftUI and MVVM."
keywords:
 - faceid
 - touchid
 - swiftui
 - swift
 - programming
 - apple
 - ios
 - ipados
---

As SwiftUI is still relatively new, and it is not clear yet for many people how to use MVVM on iOS, I decided to write this short article in which I explain how one would integrate Face ID/Touch ID with SwiftUI.

Let's remember that SwiftUI uses the MVVM design pattern over the traditional MVC, and this can be confusing for people who are migrating to the new pattern for the first time. That said, the main takeaway from this article is to understand that views get destroyed and rebuilt *very* often in SwiftUI, and therefore the right place to write this kind of logic is in the `ViewModel`

**Note**: This article will explain how to use the basic local authentication APIs to show a practical example of how it can be done. Don't use this in a real sensitive application. If you need to add actual security, you can make use of what you learn in this article alongside my other article titled [Using the iOS Keychain with Biometrics](https://www.andyibanez.com/posts/ios-keychain-touch-id-face-id/) to integrate the authentication APIs with the Keychain APIs.

# Project Setup

The first thing you need to do is to add the `NSFaceIDUsageDescription` key to your `Info.plist` with a string explaining why your app needs Face ID. If you don't set this key, your app is going to crash before your app has a chance to show the authorization prompt.

## The Authorization Code

If you have been googling to use Face ID/Touch ID in your app, you have likely come across similar code to this:

```swift
  func requestBiometricUnlock() {
    let context = LAContext()
    
    var error: NSError? = nil
    
    let canEvaluate = context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error)
    
    if canEvaluate {
      if context.biometryType != .none {
        print("We got a biometric")
        context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "To access your data") { (success, error) in
          if success {
            print("Authenticated successfully!")
          }
        }
      }
    }
  }
```

And you have tried to integrate it in your SwiftUI just to find out that it doesn't work or it behaves very erratically. Remember that SwiftUI uses MVVM and it relies on state management to do anything useful.

The good news is that this very same code works with SwiftUI, with some modifications.

## Creating the View

We are going to create a very simple app. We are going to simulate a simple app with sensitive data and we will add another screen prompting users to login.

The first screen will consist of a tab bar. and some secret data.

```swift
struct MainApp: View {
    var body: some View {
        TabView {
            Text("Secret Page one")
                .tabItem {
                    Label("My Secrets", systemImage: "lock.doc")
                }
            
            Text("Secret page two")
                .tabItem {
                    Label("Your secrets", systemImage: "lock.square")
                }
        }
    }
}
```

Go to your main app file (the one with your `WindowGroup`), and call include your view in the hierarchy.

```swift
@main
struct Touch IDswiftuiApp: App {
    var body: some Scene {
        WindowGroup {
            MainApp()
        }
    }
}
```

When you build and run your app, you should see this:

![Main App Screen](/img/Face ID_Touch ID_swiftui_main.png)

With this, we have a full functional app without Face ID login. We are going to add the Face ID part now, and we will start by designing a simple Face ID screen.

```swift
// Face IDLoginView.swift
struct Face IDLoginView: View {
    @Binding var appUnlocked: Bool

    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: "Face ID")
                .resizable()
                .frame(width: 150, height: 150)
            
            Button(action: {
                print("Prompt Face ID login")
            }, label: {
                HStack {
                    Spacer()
                    Text("Login now")
                        .fontWeight(.bold)
                    Spacer()
                }
                .padding(10)
                .background(Color.blue)
                .foregroundColor(.white)
                .clipShape(RoundedRectangle(cornerRadius: 12))
            })
        }
        .padding()
    }
}
```

![Face ID Login Screen](/img/Face ID_swiftui_login_view.png)

These are the only two views we need for this simple. We will now setup the logic to allow Face ID to unlock the app later. We will also add a very temporary logic to ensure our login view looks properly when running it on the simulator.

We will use that `@Binding` variable (appUnlocked) to control a boolean passed from somewhere else. For now, make your button change the value of this variable to `true`.

```swift
Button(action: {
    appUnlocked = true // Unlocking the app by tapping a button.
}, label: {
    HStack {
        Spacer()
        Text("Login now")
            .fontWeight(.bold)
        Spacer()
    }
    .padding(10)
    .background(Color.blue)
    .foregroundColor(.white)
    .clipShape(RoundedRectangle(cornerRadius: 12))
})
```

Now, go back to your `@main` and make it look like this:

```swift
@main
struct Touch IDswiftuiApp: App {
    @State var appUnlocked = false
    
    var body: some Scene {
        WindowGroup {
            ZStack {
                if appUnlocked {
                    MainApp()
                } else {
                    Face IDLoginView(appUnlocked: $appUnlocked)
                        .background(Color.white)
                }
            }
        }
    }
}
```

What we are doing here is to conditionally show the main app view or the login view depending on the status of the `appUnlocked` variable.

**Note**: You may have noticed that I have a ZStack rather than just switching the view directly within the WindowGroup. The reason for this is that, I don't know if this is a SwiftUI bug, but even in a ZStack, the tab bar of `MainApp` will show up in `Face IDLoginView`, which is not something we want here. I'm keeping the ZStack to give you space to animate transitions between the views.

If you run the app now you will notice that you can tap the button and the views will swap. This is a very good starting point, and we can work on top of this to add the Face ID integration.

## The View Model

We can finally integrate the ViewModel, which will handle the Face ID logic for us. You have basically two ways of doing this:

1. You can create a ViewModel for the `Face IDLoginView` view and update the binding inside your view with a completion handler.
2. Create a ViewModel for the App itself (I'd call it `AppContext`), and pass the context itself to any views that need it, either as an `@EnvironmentObject` or `@ObservedObject` and have the Login view modify the state within it. I prefer this approach as I'd consider `appUnlocked` to be global state, and many views could, presumably could depend on it.

I will go with the second approach is it makes more sense with our particular context.

This is what our `AppContext` looks like:

```swift
// AppContext.swift
import Foundation
import SwiftUI
import LocalAuthentication

class AppContext: ObservableObject {
    @Published var appUnlocked = false
    @Published var authorizationError: Error?
    
    func requestBiometricUnlock() {
        let context = LAContext()
        
        var error: NSError? = nil
        
        let canEvaluate = context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error)
        
        if canEvaluate {
            if context.biometryType != .none {
                context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: "To access your data") { (success, error) in
                    DispatchQueue.main.async {
                        self.appUnlocked = success
                        self.authorizationError = error
                    }
                }
            }
        }
    }
}
```

We will move our `appUnlocked` state from the `App` itself into our topmost `ViewModel` called `AppContext`. `requestBiometricUnlock()` is the same method I showed you above, but I have tweaked it a bit to fit MVVM better. I have also added a `authorizationError` variable. We are not going to use it in this article, but feel free to use if you want to react or simply show any error that occurs to your users.

Now, we need to do some modifications to ` Face IDLoginView` to call the context's `requestBiometricUnlock()` method and modify our `appUnlocked` variable accordingly. Once this variable is updated, so will our view hierarchy.

```swift
struct Face IDLoginView: View {
    @ObservedObject var appContext: AppContext
    
    var body: some View {
        VStack(spacing: 24) {
            Image(systemName: "Face ID")
                .resizable()
                .frame(width: 150, height: 150)
            
            Button(action: {
                appContext.requestBiometricUnlock()
            }, label: {
                HStack {
                    Spacer()
                    Text("Login now")
                        .fontWeight(.bold)
                    Spacer()
                }
                .padding(10)
                .background(Color.blue)
                .foregroundColor(.white)
                .clipShape(RoundedRectangle(cornerRadius: 12))
            })
        }
        .padding()
    }
}
```

Finally, modify your app's entry point to use the `AppContext` as a `@StateObject` and pass in this object to `Face IDLoginView`.

```
@main
struct Touch IDswiftuiApp: App {
    @StateObject var appContext = AppContext()
    
    var body: some Scene {
        WindowGroup {
            ZStack {
                if appContext.appUnlocked {
                    MainApp()
                } else {
                    Face IDLoginView(appUnlocked: $appContext.appUnlocked)
                        .background(Color.white)
                }
            }
        }
    }
}
```

You could use an `@EnvironmentObject` instead of `@ObservedObject`, but I prefer to use `@EnvironmentObject` when there is a deep hierarchy of views spread across different files. Since in this case we only have one level "deep", I decided to use `@ObservedObject`, but feel free to experiment with `@EnvironmentObject` if you want.

And with this, our app works. We can login with Face ID without an issue now. On the simulator, feel free to go to Hardware > Face ID to simulate Face ID and successful face reads. It's useful when testing apps that require on biometric unlock on iOS.

# Sample Project.

You can delete a sample project from [here](/archives/Touch IDswiftui.zip). It has a little bonus, which is transition animations when the Face ID scan is successful.

# Conclusion

Integrating SwiftUI with Face ID and Touch ID is not at all complicated. Really, once you understand MVVM and how it plays out in the SwiftUI world, you will be able to integrate almost anything with SwiftUI. If you are still struggling to understand how MVVM works, maybe my [Using CoreLocation with SwiftUI](https://www.andyibanez.com/posts/using-corelocation-with-swiftui/) article will help you.