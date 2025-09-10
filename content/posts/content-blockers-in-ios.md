---
title: "Writing Content Blockers for iOS"
date: 2020-05-20T07:00:00-04:00
publishDate: 2020-05-20T07:00:00-04:00
originalDate: 2020-05-17T22:32:37-04:00
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
description: "Learn how to write Content Blockers for iOS."
keywords:
 - swift
 - programming
 - apple
 - ios
 - macos
---

A few years ago, Apple introduced the Content Blocking APIs to Safari. Using these APIs, developers are able to write extensions that allow Safari to block content users do not want to see.

Most commonly used for ads, content blockers are not really *ad blockers*. While they can, and commonly do, block ads, developers can write content blockers for all kind of content, including profanity, or other questionable content.

# Content Blocking VS Ad Blocking.

Content Blockers are actually very limited in terms of what they can do. We cannot really achieve the level of functionality especialized extensions such as AdBlock Plus have to offer. Apple' implementation has two main focuses in mind:

* Privacy
* Performance

## Privacy

When we write content blockers, we are just writing a set of rules that we hand to Safari, and then Safari takes care of the blocking based on those rules. The app that handed the browser these rules will never get any callbacks or notifications back from Safari when it finds content it needs to block. It is set-and-forget.

This has some privacy implications. First, developers have no access whatsoever to anything Safari blocks. There are some browser extensions that allow you to see statistics about blocked websites on the desktop. Because we don't have this level of control, we cannot really implement a feature similar to this.

Developers cannot track the content users block, and this goes in hand with Apple's policies on user data.

## Performance

Normal extensions that run in a browser have more access to resources, because they are packed with features. Because content blockers can't really execute any code on their own, the performance is blazing fast. Safari's parsing of these rules is incredibly efficient, and users are unlikely to see a negative impact on performance from this system.

Fun little fact: Content blockers were introduced in iOS 9, but only devices that had a 64-Bit processor could execute them. This means anything starting on the A7. The iPhone 5, while it did support iOS 9, it wasn't able to run content blockers.

# Writing Content Blockers

Here's the fun bit about content blockers:

**You don't need any actual code at all**

Yep. You read that right. At the most elemental level, content blockers are rules represented in a `JSON` file. Everything all the Content Blocker apps in the App Store do is create a JSON file that gets handed to Safari. You could theorically create an extensions that all it does is hand a static JSON file to Safari without writing a line of code.

If you create a new `Content Blocking Extension` on Xcode, the target has a `blockList.json` file. If you open it, it has content like this:

```json
[
    {
        "action": {
            "type": "block"
        },
        "trigger": {
            "url-filter": "webkit.svg"
        }
    }
]
```

The extension does provide a Swift file called `ContentBlockerRequestHandler.swift`. This file has the bare minimum to create and work with your extension. You don't need to touch it at all, but know that you can use this file to customize your content blocker.

## The blockList.json

Back to the meaty parts of content blockers, the JSON file lets you add all the rules you want to use for blocking. And they are quite configurable, too. The JSON has two main keys:

`action`: This is the action you want to perform on content that match the trigger.
`trigger`: This is the condition that should be met in order to execute your `action`.

### Triggers

The trigger can be a combination of:

* `url-filter`: This matches the URL in the navigation bar of the browser. It can be a regular expression.
* `url-filter-is-case-sensitive`: You can decide if the URL matching should be case sensitive. By default, it is case insensitive.
* `resource-type`: This is an array that let's you specify what kind of content should be blocked. Some valid values are `script` and `image`, but there's others.
* `load-type`: Is another array, you can specify if you want to let the website load its own content (`first-party`) and/or content from other URLs (`third-party`)
* `if-domain` and `unless-domain` let you specify a website to include or exclude from the rule.

### Actions

Actions describe how the blocking should take place.

* `block`: Blocks a content entirely, rewriting the HTML.
* `block-cookies`: allows you to block the cookies set by certain requests.
* `css-display-none`: Modifies the CSS to hide the content. When using this type, you need to provide a CSS selector (with the `selector` JSON property) to target the CSS you want to modify.
* `ignore-previous-rules`: You can override previous rules if they match a certain criteria.

## Writing Simple Content Blockers.

With that knowledge in hand, you could block Google and a bunch of their services writing a code like this:

```swift
[
    {
        "action": {
            "type": "block"
        },
        "trigger": {
        "url-filter": ".*google.*"
        }
    }
]
```

This simple rule will prevent Google from loading at all, and any URL that has `google` in it won't load, either.

If you wanted to block content that leads to Buzzfeed, you would write something like this:

```swift
[
    {
        "action": {
            "type": "css-display-none",
            "selector": "a[href*='buzzfeed']"
        },
        "trigger": {
        "url-filter": ".*"
        }
    }
]
```

This will block all Buzzfeed content, across all URLS.

# Conclusion

Content blockers are an old but very powerful extension. They allow us to write performant rules for blocking content in Safari while respecting our user's privacy.

