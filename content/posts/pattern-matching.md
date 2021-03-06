---
title: "Introduction to Patterns and Pattern Matching in Swift."
date: 2019-10-30T07:00:00-04:00
originalDate: 2019-10-25T15:57:05-04:00
draft: false
publishDate: 2019-10-30T07:00:00-04:00
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
 - patternmatching
categories:
 - development
description: "Learn how to apply pattern matching with Swift in your code."
keywords:
 - swift
 - patternmatching
 - ios
 - tvos
 - ipados
 - watchos
---

Swift is a beautiful language, but it hides some powerful features from developers who come from more "old-style" programming languages such as C++ and Java. One such feature is Pattern Matching, and it allows you to write some cleaner code when dealing with some operations.

For example, consider casting. Casting is a feature in the vast majority of statically-typed languages. Casting is considered to be an ugly operation by some, because when you need to cast, it's usually because the language has a flaw that prevents it from telling you about the right data type underneath. This is specially true when you add in Object-Oriented Programming and classes are marked to return a super type instead of a specific subtype. With pattern matching, you can more cleanly check for datatypes without having to worry about crashes or weird behavior.

In this article, we will learn what "Patterns" are in this context, we will explore the specific casting pattern, and we will explore a few other common patterns that are used in Swift. We will also talk about pattern matching along the way so you can learn about this feature along the way, if you haven't already.

# Introduction to Patterns and Pattern Matching

Pattern matching is the checking and locating of specific sequences of data in some specific pattern mixed with random data. Not to be confused with pattern recognition, which allows us to observe a bunch of raw data and find some kind pattern into it. In pattern matching, you already know what the pattern looks like, and you want to know when it's found, and what kind of data to retrieve from it, if applicable.

The pattern represents the structure of a single value or a composite value. You can essentially use this pattern to get information about data, and/or the data itself. Patterns can be very varied, from data type checking to enum checking, and more.

In Swift, pattern matching is usually done with the `switch` statement, but you can use this powerful feature with other features as well, like `for` loops and even `catch`.

# Types of Patterns

Switch has the following patterns:

* Wildcard Pattern
* Identifier Pattern
* Value-Binding Pattern
* Tuple Pattern
* Enumeration Case Pattern
* Optional Pattern
* Type-Casting Pattern
* Expression Pattern

We will explore all of them and learn how to apply them in Pattern Matching.

## The Wildcard Pattern

The Wildcard Patterns ignores any value and replaces it with *nothing*, represented as an underscore. You can use this pattern when you don't care about the values provided in a pattern.

In the following example, you can do something three times with a `for` loop. Swift doesn't have the traditional `for(;;)`  loop, so if you need to do something repeatedly without necessarily caring about a value, you can do this:

```swift
for _ in 1...3 {
  print("I'm doing something!")
}
```

You commonly use the `for` loop to fast-iterate over the values of a collection. This works, because `1...3` creates a [`Range`](https://developer.apple.com/documentation/swift/Range) which on every iteration returns a number within it. We are *matching* any random in the range and using an underscore to represent it, since we don't care about its value.

## Identifier Pattern

This is the most common pattern of all, and you use it all the time. This pattern matches any value and binds it to a variable or constant name. It's a full mouth way of saying "variable assignment".

```swift
let doll = "Classical Alice"
```

This will assign the value "Classical Alice" to the variable "doll".

You have used this pattern before, but did you know you can discard the contents of the assignment? This is useful when you call a function with side effects and you don't need to store its value at all. I do this all the time with `Sec` functions that return a OSStatus, but there's situations when it may never return any errors at all:

```swift
let _ = functionWithSideEffects()
```

## Value-Binding Pattern.

In this pattern is where things start to become more interesting. This pattern allows you to match a tuple and grab its values independently.

```swift
let coordinate = (5, -3)
switch coordinate {
case let (x, y):
  print("Coordintate: \(x), \(y)")
}
```

Also, now is a good time to mention that you can use the `where` keyword to do more complex pattern matching. In the following example, we will not bind the `y` value (we are replacing it with an underscore), and we will only match the `x` value when it is higher than 10:

```swift
let coordinate = (5, -3)
switch coordinate {
case let (x, _) where x > 10:
  print("This coordinate is way to the right of the x axis. \(x)")
}
```

It's also worth noting that the `case` match in the order they appear in. Consider the following:

```swift
let coordinate = (12, -3)
switch coordinate {
case let (x, y):
  print("Coordintate: \(x), \(y)")
case let (x, _) where x > 10:
  print("This coordinate is way to the right of the x axis. \(x)")
}
```

Despite the value we want has a value bigger than 10, the second case will never be executed. This is because the first case matches perfectly. You can, however match multiple patterns with the `fallthrough` keyword.

```swift
let coordinate = (12, -3)
switch coordinate {
case let (x, y):
  print("Coordintate: \(x), \(y)")
  fallthrough
case let (x, _) where x > 10:
  print("This coordinate is way to the right of the x axis. \(x)")
}
```

You can write very interesting and easy to understand code with this.

## Tuple Pattern

A tuple is a comma-separated list of zero or more patterns. You may have used patterns before:

```swift
func makeCoordinate(x: Int, y: Int) -> (Int, Int) {
	return (x, y)
}
```

Tuples are very powerful and they extend the power of pattern matching even more.

Consider an array of Coordinates. You can bind a value in each iteration to get the values.

```swift
let coordinates = [(1, 1), (2, 2), (5, 5), (7, 5), (9, 2), (3, 5)]
for (x, y) in coordinates {
  // Only iterate over the values that have a 5 in the `y` position.
}
```

But even more interesting, once again, you can use the `where` keyboard to further constraint the matching. In the following example we will only get the coordinates that have a value of `y > 5`.

```swift
let coordinates = [(1, 1), (2, 2), (5, 5), (7, 5), (9, 2), (3, 5)]
for (x, y) in coordinates where y > 5 {
  // Only iterate over the values that have a 5 in the `y` position.
}
```

## Enumeration Case Pattern

This is a very nice pattern, and a very powerful one. This is the kind of pattern you can use with `Error`. This kind of patterns lets not only match pure cases, but also cases with parameters.

```swift
enum DataError: Error {
  case writingError(localizedDescription: String, file: Data)
  case readingError(localizedDescription: String, fileURL: URL)
  case unknownError
}

let error = DataError.writingError(localizedDescription: "Error writing file", file: Data())
switch error {
case .writingError(let description, let data):
  print("Could write data \(data) because \(description)")
case .readingError(let description, let url):
  print("Couldn't read file at \(url) because \(description)")
case .unknownError:
  print("Unknown error")
}
```

We defined a `DataError` and we can match all the different cases. And yes, in case of the case parameters (such as `localizedDescription`), you can use the wildcard pattern if you don't care about retrieving their values at all.

## Optional Pattern

In Swift, Optionals are just syntactic sugar for the Optional Pattern. An optional is simply an `enum`, and you can match this even.

Take a look at the following code:

```swift
let anOptional: Int? = 50
if case .some(let value) = anOptional {
  // The optional has a value
  print(value)
}
```

In Swift, we can use this shorthand form:

```swift
if case let value? = anOptional {
  print(value)
}
```

But the real power of this pattern comes from other uses. You can use it to iterate over an array of options and ignore the values that are nil, for example.

```swift
let names: [String?] = ["Alice", nil, "Eileen", "Margarethe", "Alura", nil, nil, "Momoko"]
for case let name? in names {
  /// Print all the names
}
```

## Type Casting Pattern

This pattern allows to avoid casting if you want more safety and it allows you to check for object types if you ever need to. There's to forms to this pattern:

* You can check if an object is of a certain type with the `is` keyword.
* You can match and cast to see if an object is of a type and cast to it immediately to use it.

The former is only available in `switch` statements. The latter gives you more flexibility and chances are you have used it before, in the form of `if let foo = myObject as? Class {}`.

Using the `is` keyword is really easy, and you can use it if you only case about the type of an object, and not about its properties or other kind of data.

In the following example, we will check if a vehicle is of a given type and let the user know what matches:

```swift
public class Vehicle {}

public class Car: Vehicle  {
  let wheels: Int
  let size: Int
  
  init(wheels: Int, size: Int) {
    self.wheels = wheels
    self.size = size
  }
}

public class Airplane: Vehicle {
  let wings: Int
  let capacity: Int
  
  init(wings: Int, capacity: Int) {
    self.wings = wings
    self.capacity = capacity
  }
}

func makeVehicle() -> Vehicle {
  return Airplane(wings: 2, capacity: 200)
}

let aVehicle = makeVehicle()
switch aVehicle {
case is Car:
  print("The vehicle is a car")
case is Airplane:
  print("The vehicle is an airplane")
default:
  print("unknown vehicle")
}
```

Now if you need to know the type of an object and do need to access its properties, you need to use the "casting" form of this pattern. Once again, you have probably used a variation of this, but I like this other form because it doesn't rely on optionals and it writes very neat code for the most part (I'm not a fan of having to specify a default case):

```swift
let vehicle = makeVehicle()
switch vehicle {
case let vehicle as Car:
  print("This car has \(vehicle.wheels) and it's size is \(vehicle.size)")
case let vehicle as Airplane:
  print("This airplane has \(vehicle.wings) wings and a capacity for \(vehicle.capacity) people")
default:
  print("Unknown vehicle")
}
```

Do note that this pattern matches both the class you specify and any subclasses of it, so make sure you order your cases accordingly.

## The Expression Pattern

This pattern represents the value of an expression. This pattern can only appear inside `switch` statements.

This pattern users the `~=`. This operator by default compares values of the same type using `==`. It can also match elements in ranges, checking to see if the value is within the range. You can implement this function for your own types.

```swift
let coord = (1, 1)
switch coord {
case (-5...5, -5...5):
  print("Both coordinates are within the same range. x is \(coord.0) and y is \(coord.1)")
default:
  print("no matches")
}
```

# Conclusion

Pattern matching is a really powerful and interesting feature. It can help you write cleaner code and there's many ways to do it. You can use pattern matching in many different places to improve the quality of your code.

