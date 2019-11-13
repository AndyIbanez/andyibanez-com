---
title: "New Search APIs in iOS 13"
date: 2019-11-06T16:32:01-04:00
publishDate: 2019-11-13T07:00:00-04:00
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
 - macos
 - tvos
 - watchos
 - uikit
 - search
 - uisearchbar
 - uisearchtoken
 - uisearchtextfield
categories:
 - development
description: "iOS 13 introduced improvements to existing UI search APIs. Learn what's changed."
keywords:
 - swift
 - ios
 - tvos
 - ipados
 - watchos
 - uikit
 - search
 - uisearchbar
 - uisearchtoken
 - uisearchtextfield
---

New Search APIs in iOS 13

iOS has always provided interesting search APIs, but they have always been limited and doing the most interesting tasks required you to either write your own implementation or use private API.

iOS 13 has provided some very nice improvement and APIs to the UI search APIs. In this article we will talk about two of them.

# UISearchBar finally exposes it's text field

I have been using `UISearchController` and `UISearchBar` for a *very* long time, and I have always found it bizarre that Apple didn't expose its underlying text field property. As of iOS 13, the search bar finally exposes it, in the form of a [`UISearchTextField`](https://developer.apple.com/documentation/uikit/uisearchtextfield) object.

With this object, you can finally customize the appearance of the search bar. You no longer need to hack away at the view hierarchy with hopes of finding the text field and customizing it there.

```swift
searchBar.searchTextField.backgroundColor = .blue
searchBar.searchTextField.textColor = .white
```

There's still some limitations, like you cannot change the bar style, but this is a very good change, and something developers have been hacking at for years. Been able to change the colors is a great improvements.

## UISearchTextFieldDelegate

Along with the text field, there's a new delegate. To tell you the truth, I have no idea what can be done with it. The [Documentation](https://developer.apple.com/documentation/uikit/uisearchtextfielddelegate) only has one [method](https://developer.apple.com/documentation/uikit/uisearchtextfielddelegate/3175446-searchtextfield) that is not documented.

# Search Tokens

Search Tokens are my absolutely favorite new feature for search in iOS 13. Some native Apple apps like mail have had it for years. While the API has existed for years, it has recently been opened up to developers.

Search tokens look like this:

![UISearchToken](/img/UISearchToken.png)

You create a search token with the [`UISearchToken`](https://developer.apple.com/documentation/uikit/uisearchtoken) class, and then you add them to the (also recently opened up) `searchTextField` properly:

```swift
let purchasesToken = UISearchToken(icon: UIImage(systemName: "tag"), text: "Purchases")
let countryToken = UISearchToken(icon: UIImage(systemName: "flag"), text: "Country")

searchBar.searchTextField.insertToken(purchasesToken, at: 0)
searchBar.searchTextField.insertToken(countryToken, at: 0)
```


The `searchTextField` lets you do more than just adding the tokens. You can change the color of the tokens (thought you cannot change them individually):

```swift
searchBar.searchTextField.tokenBackgroundColor = .blue
```

You can allow or prevent the user from copying and deleting tokens:

```swift
searchBar.searchTextField.allowsCopyingTokens = true
searchBar.searchTextField.allowsDeletingTokens = true
```

And then there's functions to get the range of one or multiple tokens, their positions, and more, so you can build powerful search features in your app.

If you need to get the tokens themselves, the text field has a `tokens` property which you can access.

```swift
    searchBar.searchTextField.tokens.forEach {
      // Do something with each search token.
    }
```

Now here's the bizarre thing: The tokens do not expose their text or image properties. Instead, you need to assign the `representedObject` property. This object is of type `Any?`, so you can assign it anything you need. Here we are assigning simple search terms:

```swift
purchasesToken.representedObject = "Purchases"
countryToken.representedObject = "Bolivia"
```

And now we can iterate over them:

```swift
searchBar.searchTextField.tokens.forEach {
  // Do something with each search token.
  print($0.representedObject)
}
```

# Conclusion

The new search APIs are really neat in iOS 13. Other than a long-overdue functionality we have been begging for for years, we also got a new beautiful tokens API, which allows to build powerful search features for our users in our apps.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.