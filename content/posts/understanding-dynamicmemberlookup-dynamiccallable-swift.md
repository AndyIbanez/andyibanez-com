---
title: "Understanding @dynamicMemberLookup and @dynamicCallable in Swift"
date: 2020-04-27T00:51:02-04:00
originalDate: 2020-04-29T07:00:00-04:00
publishDate: 2020-04-29T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - dynamism
 - swift-evolution
 - dynamiccallable
 - dynamicmemberlookup
description: ""
keywords:
 - swift
 - programming
 - apple
 - dynamism
 - swift-evolution
 - dynamiccallable
 - dynamicmemberlookup
---

If you have written code in a programming language such as Python or PHP, you can find many direct comparisons to Swift. For one, Swift is statically typed, whereas PHP and Python are not - Swift is considered a safe language as it has a bunch of features to protect you against mistakes - static typing, error throwing, optionality for dealing with nulls, to name a few -, whereas PHP and Python do not.

Swift started adding features that allows it to behave closer to the behavior of more dynamic languages. The long term implication is that this allows Swift to work alongside such languages without too many downsides.

In this article we will talk about two features Swift has added in the last two major versions to make this possible, and how they work without necessarily weakening Swift's features that ultimately make it a safe language.

# Accessing Member Properties with @dynamicMemberLookup

Introduced in Swift 4.2, Swift introduced `@dynamicMemberLookup`. Marking your objects with this class will require them to a implement a subscript with the following signature:

```swift
subscript(dynamicMember member: String) -> AnyObject
```

Where `AnyObject` can be any type. `@dynamicMemberLookup` allows you to specify any property in your class - even if it doesn't exist -, and return a value for it.

Consider the following example:

```swift
@dynamicMemberLookup
class Country {
  subscript(dynamicMember member: String) -> String {
    let properties = ["name": "Bolivia", "location": "South America"]
    return properties[member, default: "NOT FOUND"]
  }
}
```

This class has no real members to speak of, but because it was marked with `@dynamicMemberLookup`, we can try to access any property we can think of. We chose this implementation to return a `String`, so Swift's type safety is still pretty valid.

```swift
let country = Country()
print(country.name)
print(country.location)
print(country.population)
```

All the property accessors will go through the `subscript` method and return a value for it. In the example above we provide a default value of the subscript to return when the expected property does not exist.

The code above will print:

```
Bolivia
South America
NOT FOUND
```

You can implement multiple `subscript` methods to return different types. In the following example we will add a new `subscript(dynamicMember)` subscript to return ints, which can be the population of a country:

```swift
class Country {
  subscript(dynamicMember member: String) -> String {
    let properties = ["name": "Bolivia", "location": "South America"]
    return properties[member, default: ""]
  }
  
  subscript(dynamicMember member: String) -> Int {
    let properties = ["population": 11_673_021]
    return properties[member, default: 0]
  }
}
```

If you try to run the new code as is:

```swift
let country = Country()
print(country.name)
print(country.location)
print(country.population)
```

Swift is actually not knowing to know what to do. Because we have more than one `subscript(dynamicMember)` now, Swift is not sure which one to call. To solve this, we need to move the properties to variables, and explicitly specify their type. The following code will compile and work as expected:

```swift
let country = Country()
let name: String = country.name
let location: String = country.location
let population: Int = country.population
print(name)
print(location)
print(population)
```

```
Bolivia
South America
11673021
```

Using `@dynamicMemberLookup` doesn't mean that you have to stop using properties altogether, consider the following:

```swift
@dynamicMemberLookup
class Country {
  let name: String
  
  init(name: String) {
    self.name = name
  }
  
  subscript(dynamicMember member: String) -> String {
    
    let properties = ["name": "Bolivia", "location": "South America"]
    return properties[member, default: ""]
  }
  
  subscript(dynamicMember member: String) -> Int {
    let properties = ["population": 11_673_021]
    return properties[member, default: 0]
  }
}


let country = Country(name: "Chile")
let name: String = country.name
let location: String = country.location
let population: Int = country.population
print(name)
print(location)
print(population)
```

```
Chile
South America
11673021
```

Swift will first try to get an actual member with the name you specify, and if it doesn't find one, then it will call the relevant `subscript` method.

Keep in that, if you find use for `@dynamicMemberLookup`, you will lose autocomplete for all the properties you expect it to cover. There's currently no way for the IDE to see missing properties, as these ones are added on the runtime.

# Marking Types as Directly Callable with @dynamicCallable

Introduced in Swift 5, `@dynamicCallable` allows your types to be *callable* types. What this means is that you can use your types directly to execute some code.

When you mark your classes with `@dynamicCallable`, you need to at least one of the following methods:

```swift
func dynamicallyCall(withArguments args: [AnyObject]) -> AnyObject

func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, AnyObject>) -> AnyObject
```

The difference is that the first one is used when you don't want to use argument labels (like `foo(1, 2)`), or if you would rather have them (`foo(firstValue: 1, secondValue: 2`).

In the case of the first function, it doesn't take an array, but rather a `ExpressibleByArrayLiteral`, so you can specify an array, a set, or anything other that conforms to it.

The following case will allow you to change the capitalization of any string. To demonstrate the use of `@dynamicCallable`, we will implement it with both signatures.

The first one doesn't label the parameters, so it's like an old C-Style call:

```swift
@dynamicCallable
class CaseChanger {
  enum Case: String, RawRepresentable {
    case uppercase = "uppercase"
    case lowercase = "lowercase"
  }
  
  func dynamicallyCall(withArguments args: [String]) -> String? {
    guard let casing = args.first, let string = args.last else {
      return nil
    }
    
    let casingType = Case(rawValue: casing.lowercased()) ?? Case.lowercase
    return casingType == .uppercase ? string.uppercased() : string.lowercased()
  }
}
```

```swift
let caseChanger = CaseChanger()
let changedCase = caseChanger("uppercase", "Katarina Claes")
print(changedCase)
```

```
Optional("KATARINA CLAES")
```

The dynamism here allows us to do a lot of manual checking, but it's part of the price we pay to get this neat dynamism. `withArguments` gives an array of the all the arguments passed to the function call so you can operate on them as needed. In this example we assume the first parameter is the casing we want, and the second one the string we want to change the case of.

The second variation lets us create dynamic calls specifying an argument label. This is the "swiftier" way.

```swift
@dynamicCallable
class CaseChanger {
  enum Case: String, RawRepresentable {
    case uppercase = "uppercase"
    case lowercase = "lowercase"
  }
  
  func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, String>) -> String? {
    guard let casingPair = args.first, let stringPair = args.last else {
      return nil
    }
    
    guard casingPair.key == "casing" && stringPair.key == "string" else {
      return nil
    }
    
    let casingType = Case(rawValue: casingPair.value.lowercased()) ?? Case.lowercase
    return casingType == .uppercase ? stringPair.value.uppercased() : stringPair.value.lowercased()
  }
}
```

```swift
let caseChanger = CaseChanger()
let changedCase = caseChanger(casing: "lowercase", string: "IT WAS MANY AND MANY A YEAR AGO")
print(changedCase)
```

```
Optional("it was many and many a year ago")
```

Of course, thanks to the dynamism we have a lot of work to do with the checking. You may want to check that you have all the required labels, that they are in the correct order, and more. Your checks can be as complex as necessary depending on your case.

This is a very powerful feature, but you should probably sty clear, depending on your needs.

# Conclusion

Starting on Swift 4.2, the language has received many updates to make interoperability with dynamic languages easier. `@dynamicMemberLookup` and `@dynamicCallable` are very powerful features, but they have a lot of considerations and you should ask yourself twice if you want to use them.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.