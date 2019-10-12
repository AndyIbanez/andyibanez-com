---
title: "Filtering Arrays with Predicates"
date: 2019-10-12T17:28:31-04:00
draft: true
draft: false
publishDate: 2019-16-09T07:00:00-04:00
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
 - nspredicate
 - arrays
 - nsarray
categories:
 - development
description: "Learn how to use the powerful NSPredicate API for searching and filtering."
keywords:
 - swift
 - nspredicate
 - arrays
 - nsarray
 - ios
 - tvos
 - ipados
 - watchos
---

Whether you have been programming for a while or are new at it, chances are you have had the need to search for results in an array. And while Apple's SDKs for iOS, macOS, iPadOS, and watchOS all use Foundation and have a set of handy tools to make that task easier, there is one particular API that is very powerful but doesn't get much use unless you pair with other frameworks such as Core Data: NSPredicate.

[`NSPredicate`](https://developer.apple.com/documentation/foundation/nspredicate) is a definition of logical conditions you can use to search and filter information in certain APIs. It lets you define conditions to search for something while using an object's properties. Internally, the API uses key-value coding to work, so your Swift objects must be compatible with Objective-C.

# Introduction to NSPredicate

NSPredicate allows you to perform searches and filtering writing conditions like the following:

* A person's salary is equal to 5000, less than 3000, more than 4000, and so on.
* A person's last name is a certain last name.
* A person's last name is LIKE (similar) to another last name.
* A text begins with a certain string.
* Date in a range.

And to make it even more interesting, you can even perform aggregate operations such as the sum of a certain property in an array, the average, and so on.

There's some subclasses of this class you can use, but you can also use it as-is. [`NSComparisonPredicate`](https://developer.apple.com/documentation/foundation/nscomparisonpredicate) can be used to compare the results of two expressions, and [`NSCompoundPredicate`](https://developer.apple.com/documentation/foundation/nscompoundpredicate) can be used to join two predicates together to create logical "and", "or", and "not" searches and filtering.

In this article we will explore `NSPredicate` and `NSCompoundPredicate` and how you can use them to search in NSArrays. You can later apply these concepts for other frameworks such as Core Data.

# Downsides and Quirks.

Before we get to work, though, I need to mention that `NSPredicates` rely heavily in Objective-C's features, so when we work on it with Swift, we lose some features such as type safety and we need to do a bit of casting. That said, this extra work is nothing considering all that you get back from them.

## Using NSPredicate

Our objects must work with Objective-C, specially it's Key-Value Coding features. We will write NSPredicate examples using these two classes:

```swift
@objcMembers class Job: NSObject {
  let company: String
  let salary: Float
  let title: String
  
  init(company: String, salary: Float, title: String) {
    self.company = company
    self.salary = salary
    self.title = title
  }
}

@objcMembers class Person: NSObject {
  let firstName: String
  let lastName: String
  let job: Job
  
  var fullName: String {
    get {
      return "\(firstName) \(lastName)"
    }
  }
  
  init(firstName: String, lastName: String, job: Job) {
    self.firstName = firstName
    self.lastName = lastName
    self.job = job
  }
}
```

In order to make them available to Objective-C, we need to mark them as `objcMembers`, and to get the KVC features, we need them to inherit from `NSObject`.

Then we will use an array that stores `Person` objects, where each person has a `Job`.

```swift
let people = [
  Person(firstName: "Andy", lastName: "Ibanez", job:
    Job(company: "Fairese", salary: 5000, title: "CEO")),
  Person(firstName: "Sakura", lastName: "Kinomoto", job:
    Job(company: "Tomoeda Gakkou", salary: 4000, title: "Card Captor")),
  Person(firstName: "Daidouji", lastName: "Tomoyo", job:
    Job(company: "Daidouji Group", salary: 4000, title: "Filmmaker")),
  Person(firstName: "Nae", lastName: "Kinomoto", job:
    Job(company: "Animal Group", salary: 3000, title: "Animal Captor")),
  Person(firstName: "Tae", lastName: "Kinoshita", job:
    Job(company: "Zombie, Co.", salary: 2500, title: "Dancer"))
]
```

Unfortunately, we can't use `NSPredicate` and its subclasses with a Swift array, so we need to convert it to an `NSArray`.

```swift
let nsPeople = people as NSArray
```

With all this setup done, we can write some examples.

### The NSPredicate Class

`NSPredicate` is very to use. When you create it with the `init(format:arguments:)` initializer, you pass in a string very similar to a SQL query.

Showing all the possible formats is beyond the scope of this article, but Apple provides a nice [Predicate Format String Syntax guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Predicates/Articles/pSyntax.html#//apple_ref/doc/uid/TP40001795). We will explore the basic scenarios, as well as some neat things you can do with them.

#### Exact String Match.

You can write predicates that simply match a string completely. You can adapt the query to make it match different case, and more.

The below code fetches all the people who have the last name `Kinomoto`:

```swift
let lastNameKinomotoPredicate = NSPredicate(format: "lastName = %@", "Kinomoto")
let lastNameKinomoto = nsPeople.filtered(using: lastNameKinomotoPredicate)

print("People whose last name is Kinomoto:")
(lastNameKinomoto as! [Person]).forEach {
  print($0.fullName)
  // Prints:
  // Sakura Kinomoto
  // Nae Kinomoto
}
```

Part of the downsides is we need to cast the filtered array in order to use the objects within it (or you could use Key-Value Coding yourself, but it's not as neat).

#### Matching The Beginning of a String

You can match any part of a string without having to match the entirety of it. To match the beginning of the string, you use the `BEGINSWITH` keyword.

The bellow example will fetch all the people's whose last name begins with `Kino`.

```swift
let lastNameBeginsKinoPredicate = NSPredicate(format: "lastName BEGINSWITH[c] %@", "Kino")
let lastNameBeginsKino = nsPeople.filtered(using: lastNameBeginsKinoPredicate)

print("People whose last name contains \"Kino\":")
(lastNameBeginsKino as! [Person]).forEach {
  print($0.fullName)
  // Prints:
  // Sakura Kinomoto
  // Nae Kinomoto
  // Tae Kinoshita
}
```

#### Matching Properties Within Composed Objects.

All our `Person`s have a `Job`. We can find all the people who meet a certain job criteria by querying the `Job` object. We can do this using a key path (which is not the neat key path's we know from Swift, but they work)!

In the example bellow, we will query for all the people who work at companies who have the word `Group` in their name:

```Swift
let companyContainsGroupPredicate = NSPredicate(format: "job.company CONTAINS[c] %@", "Group")
let companyContainsGroup = nsPeople.filtered(using: companyContainsGroupPredicate)

print("People who work for a group:")
(companyContainsGroup as! [Person]).forEach {
  print($0.fullName + " " + "(\($0.job.company))" )
  // Prints:
  // Daidouji Tomoyo (Daidouji Group)
  // Nae Kinomoto (Animal Group)
}
```

This one of my favorite things about NSPredicate, and you can use it to build some advanced searches.

### Using NSCompoundPredicate

You can create and join predicates together using the `NSCompoundPredicate` class. This class inherits from `NSPredicate`, so you can use it everywhere a normal predicate is expected.

You can use a compound predicate to join various predicates together with "and", "or", and even "not". You can also use other compound predicates in another compound predicate, so you can create very interesting and complex filters.

In the example below we will create a predicate that fetches all the people who earn less than 3000 and those who earn above 4000 with a `or` predicate.

```swift
let salaryBelow3000 = NSPredicate(format: "job.salary < %d", 3000)
let salaryAbove4000 = NSPredicate(format: "job.salary > %d", 4000)
let salaryBelow3000AndAbove4000Predicate = NSCompoundPredicate(orPredicateWithSubpredicates: [salaryBelow3000, salaryAbove4000])
let salaryBelow3000AndAbove4000 = nsPeople.filtered(using: salaryBelow3000AndAbove4000Predicate)

print("People who earn less than 3000 and above 4000:")
(salaryBelow3000AndAbove4000 as! [Person]).forEach {
  print($0.fullName + " " + "(\($0.job.salary))")
  // Prints:
  // Andy Ibanez (5000.0)
  // Tae Kinoshita (2500.0)
}
```

## Why Use NSPredicate?

This is a very good question, and you may be wondering since we already have a lot of powerful filtering options in Swift using the `filter` function. We can do everything we did with predicates using this function and save us the problem of making our code compatible with Objective-C and all that casting. So why?

The answer is that, not only is this great for using with Core Data (where you specify the criteria with NSPredicates) but also because it's much easier to allow your users to create filters for their data. Imagine you create a Contacts app that doesn't store its data in a database, but in plain text files that can be deserializes into arrays. You can allow your users to create filters for their contacts with any criteria they want. Of course, only use them if you have such a need. If you need to do filtering that your users are never going to see, there's no need to use NSPredicates with arrays.

# Conclusion

NSPredicate offer a very powerful way to search and filter array and other content, such as queries in core data. They can be as simple or complex as you need them, and their format language is similar to SQL and therefore very intuitive to use. It's a bit of a bummer we need to play with a lot of Objective-C code to make them to work, but if you need them, the effort is worth it.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.