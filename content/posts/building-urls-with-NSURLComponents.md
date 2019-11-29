---
title: "Building URLs With NSURLComponents"
date: 2019-09-04T7:00:00-04:00
originalDate: 2019-09-04T14:00:00-04:00
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
 - tvos
 - watchos
 - ipados
 - urls
 - networking
 - nsurl
categories:
 - development
description: "Learn to build URLs Swiftly with NSURLComponents."
keywords:
 - swift
 - urls
 - networking
 - ios
 - tvos
 - ipados
 - watchos
---

If you have been programming for Apple platforms for a while, chances are you have seen (or maybe even wrote yourself) a line of code that looks like this:

```swift
let url = URL(string: "https://www.google.com/search?hl=en&q=pullip")!
```

Whether you wrote it yourself or someone else did it, one thing is clear: This is not a safe way to build URLs. Can you know, for sure, that your URL is actually valid? Intuitively, all of us can see a URL and see if it's valid, but there is a [whole lot of governing](http://www.faqs.org/rfcs/rfc1738.html) in the URL format that at some point we may find funny URLs that look valid and aren't, or the other way around; they look invalid, but aren't.

In this article we will explore a very handy class for both URL composition and decomposition: `NSURLComponents`.

# Introduction to NSURLComponents

Calling web APIs is a very common task that the vast majority of developers are going to do at some point. To do that, the iOS/iPadOS, macOS, TVOS, and watchOS SDKs have a nice set of tools that make networking easier. The process of creating a URL seems obvious with the simple `init(string:)` constructor of `NSURL`, but a URL has many more components, and it may be easier to write a bit more code to ensure you actually have a valid URL.

`NSURLComponents` is a class that allows us to both build URLs and decompose them into their essential parts. Using it, we can safely build valid URLs by just passing in the parts (components) we need. Other than making it easier to build URLs, it also handles some encoding details for us, so we don't have to care about escaping characters ourselves, especially in query strings.

## Building URLs

Let's go back to the old URL we used as an example at the beginning of this article.

```swift
let url = URL(string: "https://www.google.com/search?hl=en&q=pullip")!
```

We will try to build this URL using `NSURLComponents`. It is a little bit more code, but the peace of mind is worth it, and it also helps us understand how URLs work a bit better, without having to refer to the entirety of RFC-1738.

First, we will build the URL up to the `/search` components, and we will be excluding the query (everything after the `?`).

```swift
var components = URLComponents()
components.scheme = "https"
components.host = "www.google.com"
components.path = "/search"
if let url = components.url {
    print("URL: \(url)")
}
```

Like we would expect, this will print `URL: https://www.google.com/search`. This an example of a very simple URL, but be aware that URLs may have some additional components (and we do not need to set all the properties in order for them to be valid). For example, not all URLs have a `path`, and we can safely ignore the `host` and `schema` too. Try deleting these properties and see what URL you get back.

There's a few additional components in URLs that we will not cover in this article, but it's worth mentioning them:

* `fragment`: If a URL contains a `#` symbol, the `fragment` is everything after it.
* `user` and `password`: Some URLs require some basic authentication to access them. We can use these properties for such credentials (i.e. `https://andy:masterpassword@www.google.com`).
* `port`: If we need to specify the port of a resource, this is the right property.
* `query`: We will actually explore this one, but not as a string component. The `query` of a URL is everything after a `?` sign.


### URLs with Queries

The query part of a URL (everything after a `?`) is composed of zero or more key-value pairs.

To build the `query` component of a URL, we could use `NSURLComponents`' `query` property, which is a string. This is a quick and dirty way of doing it, but we lose some safety and if we use this, we might as well not use `NSURLComponents` at all.

```swift
components.query = "hl=en&q=pullip"
```

What we should use instead is use another class whose only purpose is to create these key-value pairs. That class is `URLQueryItem`, and it's very easy to use. It has one simple constructor which takes the `name` of the parameter (the "key"), and the value of it.

```swift
let hl = URLQueryItem(name: "hl", value: "en")
let q = URLQueryItem(name: "q", value: "pullip doll")

components.queryItems = [hl, q]
```

We all love Swift, so let's take a moment to make this Swiftier:

```swift
components.queryItems = ["hl": "en", "q": "pullip"]
    .map { URLQueryItem(name: $0, value: $1) }
```

With this, we can now build our query strings with the certainty that they are going to be valid when building URLs.

# Decomposing URLs Into Their Components

`NSURLComponents` can be used for both composing and decomposing URLs. This is very useful, specially if we use the URL mechanism Apple has in place for app communication.

One such example of this is iOS with its URL app launching mechanism. Our apps can register `scheme`s to launch other apps, and also other apps can launch ours if they register our scheme.

When an app launches ours, this application delegate method is called:

```swift
application(_:url:options:)
```

And we can use this to perform certain tasks in our app.

We could decide that an app calling this URL is going to print the passed text into the console:

`myapp://print?text='Hello+Alice'`

You could try to grab the underlying string of the passed `NSURL`, splitting the string with a token and parsing the components ourselves. But this takes a lot of work, is error prone, and there's no need to do it since `NSURLComponents` can do it for us.

In the following code, we will decompose the above URL with `NSURLComponents`, grab the `text` query parameter, and prints its value.

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    let components = URLComponents(url: url, resolvingAgainstBaseURL: true)
    components?.queryItems?.forEach {
        // In case our app grows and we start passing in bigger queries with the URLs, we will check we grab the ones we need and work with each parameter as required.
        if $0.name == "text" {
            print($0.value ?? "")
        }
    }
    return true
}
```

# Conclusion

While `NSURLComponents` does require more code to write, using it has a lot of benefit. Automatic encoding, security, and peace of mind are some of the benefits it brings to us when building URLs. Being able to decompose existing URLs into their components is also a useful thing, especially on systems where app communication through URLs is prevalent.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.