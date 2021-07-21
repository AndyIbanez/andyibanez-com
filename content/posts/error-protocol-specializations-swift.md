---
title: "Error Protocol Specializations in Swift"
date: 2020-06-18T22:00:00-04:00
draft: false
originalDate: 2020-06-14T11:27:44-04:00
publishDate: 2020-06-18T07:00:00-04:00
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
 - error handling
categories:
 - development
description: "Learn about the different Error Protocol Specializations in Swift."
keywords:
 - swift
 - error handling
 - ios
 - tvos
 - ipados
 - watchos
---

Earlier this week I was scrolling through my Twitter feed as usual and I found [this tweet](https://twitter.com/harlanhaskins/status/1270399151730118656?s=20) that made me realize I may have been handling errors incorrectly in Swift all my life. This prompted me to research a bit more about error handling in Swift, and it turns out there's many specialized `Error` protocols you can conform to, and you should probably be using them over the default `Error` provided by the language. All these specializations conform to `Error` themselves. In this article, we will explore a few specializations we can use when dealing with errors in Swift.

Keep in mind that they are part of the Foundation framework though, so they may not work when used outside Apple's platforms.

## LocalizedError

[`LocalizedError`](https://developer.apple.com/documentation/foundation/localizederror) provides four properties to display your user information about errors in their native language.

The four properties are the following and they are all strings. They are required, but they provide a default implementation.

* `errorDescription`
* `failureReason`
* `helpAnchor` - I wasn't able to find what this one is for, specifically on iOS.
* `recoverySuggestion`

A sample of implementation could be:

```swift
enum NetworkError: LocalizedError {
  case noNetwork
  case unexpectedResponse
  
  var errorDescription: String {
    switch self {
    case .noNetwork: NSLocalizedString("No network connection found", comment: "")
    case .unexpectedResponse: NSLocalizedString("The server returned an unexpected response", comment: "")
    }
  }
  
  var failureReason: String? {
    switch self {
    case .noNetwork: NSLocalizedString("Could not connect to the internet", comment: "")
    case .unexpectedResponse: NSLocalizedString("The server is not working properly", comment: "")
    }
  }
  
  var recoverySuggestion: String? {
    switch self {
    case .noNetwork: NSLocalizedString("Check your internet connection and try again", comment: "")
    case .unexpectedResponse: NSLocalizedString("Contact support", comment: "")
    }
  }
}
```

There's another neat detail about this type of error, and that is that, when bridged to Objective-C (or casted as `NSError`) all the properties of the protocol become keys of the `NSError`'s `userInfo` dictionary:

* `NSLocalizedDescriptionKey` for `errorDescription`
* `NSLocalizedFailureReasonErrorKey` for `failureReason`
* `NSLocalizedRecoverySuggestionErrorKey` for `recoverySuggestion`
* `NSHelpAnchorErrorKey` for `helpAnchor`

So if you intended your errors to bridge to Objective-C, this is one specialization to consider.

## RecoverableError

[`RecoverableError`](https://developer.apple.com/documentation/foundation/recoverableerror) provides facilities to help your users attempt to recover from errors. This specialization provides one property and two methods:

* `recoveryOptions`: This is an array of strings that you can show your user when attempting to recover from errors. This property is required and you are not provided with a default implementation.
* `attemptRecovery(optionIndex:) -> Bool`: Use this to try to recover from an error, and then return a Boolean indicating whether the operation was successful or not. The `optionIndex` corresponds to the index of the option in the `recoveryOptions` array. This is required, but you are provided with a default implementation.
* `attemptRecovery(optionindex:completionHandler:) -> Void`: Just like the previous method, use this to try to recover from the error. The difference is you use a closure to pass in the result of the recovery, so you can use this when you need to try to recover using asynchronous operations.

A quick sample implementation:

```swift
enum NetworkError: RecoverableError {
  case noNetwork
  case unexpectedResponse
  
  var recoveryOptions: [String] {
    switch self {
    case .noNetwork: return [
      NSLocalizedString("Retry", comment: ""),
      NSLocalizedString("Open Settings to Change Network", comment: "")
      ]
      
    case .unexpectedResponse: return [
      NSLocalizedString("E-Mail support", comment: ""),
      NSLocalizedString("Change server", comment: "")
      ]
    }
  }
  
  func attemptRecovery(optionIndex recoveryOptionIndex: Int) -> Bool {
    switch self {
    case .unexpectedResponse:
      if recoveryOptionIndex == 0 {
        // Mail support
      } else if recoveryOptionIndex == 1 {
        // Change server
      }
      return true
    default: return false
    }
  }
  
  // ...
}
```

Once again the properties can be accessed via `NSError`'s `userInfo`, by using the `NSLocalizedRecoveryOptionsErrorKey` for the `recoveryOptions` and the `NSRecoveryAttempterErrorKey` key to access the recovery options.

## CustomNSError

Finally, the [`CustomNSError`](https://developer.apple.com/documentation/foundation/customnserror) specialization provides us with properties to create a well-known `NSError` object, that has an error domain, error code, and the user info. All the properties are required, but you are provided with a default implementation for each:

* `errorDomain`: If you worked with Objective-C, you know this one. It's the domain of the error, in reverse DNS notation.
* `errorCode`: An error code, as an int.
* `errorUserInfo`: The `userInfo`, as a `[String: Any]` dictionary.

```swift
enum NetworkError: CustomNSError {
  enum ErrorCode: Int {
    case noNetwork
    case unexpectedResponse
  }
  
  var errorDomain = "com.andyibanez.com.myApp.NetworkError"
  var appErrorCode: ErrorCode
  
  var errorCode: Int {
    return self.appErrorCode.rawValue
  }
  
  var errorUserInfo: [String : Any] {
    let dic = [
      "URL": //...,
    ]
  }
}
```

Also bridged to `NSError`, this is the "rawest" error I could find that can be bridged to Objective-C. Since you provide the `userInfo` yourself, you don't have to worry about about what keys it has. It has a lot of flexibility, but more complicated to work with.

## Mashing Them Together

Remember that these are protocols, and you are allowed to conform to more than one protocol at the same time, so nothing prevents you from, say, creating a recoverable localized error.

```swift
enum NetworkError: LocalizedError, RecoverableError {
  case noNetwork
  case unexpectedResponse
  
  // MARK: - LocalizedError
  
  var localizedDescription: String {
    switch self {
    case .noNetwork: return NSLocalizedString("No network connection found", comment: "")
    case .unexpectedResponse: return NSLocalizedString("The server returned an unexpected response", comment: "")
    }
  }
  
  var failureReason: String? {
    switch self {
    case .noNetwork: return NSLocalizedString("Could not connect to the internet", comment: "")
    case .unexpectedResponse: return NSLocalizedString("The server is not working properly", comment: "")
    }
  }
  
  // MARK: - RecoverableError
  
  var recoverySuggestion: String? {
    switch self {
    case .noNetwork: return NSLocalizedString("Check your internet connection and try again", comment: "")
    case .unexpectedResponse: return NSLocalizedString("Contact support", comment: "")
    }
  }
  
  var recoveryOptions: [String] {
    switch self {
    case .noNetwork: return [
      NSLocalizedString("Retry", comment: ""),
      NSLocalizedString("Open Settings to Change Network", comment: "")
      ]
      
    case .unexpectedResponse: return [
      NSLocalizedString("E-Mail support", comment: ""),
      NSLocalizedString("Change server", comment: "")
      ]
    }
  }
  
  func attemptRecovery(optionIndex recoveryOptionIndex: Int) -> Bool {
    switch self {
    case .unexpectedResponse:
      if recoveryOptionIndex == 0 {
        // Mail support
      } else if recoveryOptionIndex == 1 {
        // Change server
      }
      return true
    default: return false
    }
  }
}
```

# Conclusion

Error handling in Swift suddenly became easier when I learned about these `Error` specializations. Though, to be honest, they are more powerful when using them in macOS rather than the smaller OSes, because macOS has APIs to which you can provide your errors and let the system manage their displaying and even their recovery. in iOS, they aren't as powerful, but they can still help us a lot to write better error handling code that works across the Foundation framework, and in the rest of the APIs.

