---
title: "Understanding Actors in the New Concurrency Model in Swift"
date: 2021-07-29T22:00:31-04:00
draft: true
---

---
title: "Understanding Actors in the New Concurrency Model in Swift"
date: 2021-08-04T07:00:00-04:00
originalDate: 2021-07-29T22:00:31-04:00
publishDate: 2021-08-04T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
categories:
 - article series
 - modern concurrency article series
 - modern concurrency in swift article series
 - development
tags:
 - swift
 - apple
 - programming
 - ios
 - concurrency
 - async
 - await
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - multithreading
description: "Learn how to use actors in Swift to isolate mutable state and make concurrency safer."
keywords:
 - swift
 - apple
 - programming
 - ios
 - concurrency
 - async
 - await
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - multithreading
---

*This article is part of my [Modern Concurrency in Swift Article Series](https://www.andyibanez.com/posts/modern-concurrency-in-swift-introduction/).*

###### Table of Contents

1. [Modern Concurrency in Swift: Introduction](/posts/modern-concurrency-in-swift-introduction/)
2. [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)
3. [Converting closure-based code into async/await in Swift](/posts/converting-closure-based-code-into-async-await-in-swift/)
4. [Structured Concurrency in Swift: Using async let](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/)
5. [Structured Concurrency With Group Tasks in Swift](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/)
6. [Introduction to Unstructured Concurrency in Swift](https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
8. **Understanding Actors in the New Concurrency Model in Swift**