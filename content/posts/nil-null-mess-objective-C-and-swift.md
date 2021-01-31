---
title: "nil-null-mess in Objective-C and Swift"
date: 2021-02-03T07:00:00-04:00
originalDate: 2021-01-30T22:15:29-04:00
publishDate: 2021-02-03T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
categories:
 - development
tags:
 - apple
 - objective-c
 - swift
 - nil
 - null
 - nsnull
 - nullability
keywords:
 - apple
 - objective-c
 - swift
 - nil
 - null
 - nsnull
 - nullability
description: "Learn the complexities of nullability in Swift and Objective-C, common bugs, and how to work around them."
---

All programmers are familiar with the concept of `nullability`. Whether something exists or not. Whether something is there or not.

Objective-C is very dynamic when it comes to dealing with nullability. All Objective-C programmers are familiar with this phrase:

> messages can be sent to nil.

Which means that `nil` itself can call methods, safely enough, without crashing.

In Swift, we have a bit more safety. We can send "messages" to `nil`, but only if they are the result of a chained optional. `nil` can only be a thing when we are working with optionals.

Dealing with nullability in Objective-C can be mess, and this mess can carry over to Swift when bridging, so we are going to talk about the different "kinds" of nullability here, with practical examples and situations I have come across the real world.

If you are thinking that dealing with nils is easy and "works as expected", I recommend you read this, *especially* if you have never touched Objective-C.

# Our old friend nil

Both Objective-C and Swift programmers are familiar with `nil`. When an object "points" to `nil`, it means it's pointing to nil.

nil is probably the essence of nothingness. When we have nil, it means we have *nothing*. This has some interesting implications, especially when working with Objective-C.

People who have been working with Objective-C may recall that they needed to terminate variadic parameter methods with a nil in order to tell the compiler we were done passing values.

```Objc
NSArray<NSString *> *array = [NSArray arrayWithObjects:@"Alice", @"Eileen", @"Sepia Alice", nil];
```

The code above is an old-style way of initializing an array with a variable number of arguments. Let's see what happens if we try to print it:

```objc
for(NSString *string in array) {
  NSLog(@"%@", string);
}
```

```
Alice
Eileen
Sepia Alice
```

It will print everything, except the `nil` - Because `nil` is not part of the array.

Things become even more interesting when you try to add multiple `nil` values to an array.

```Objc
NSArray<NSString *> *array = [NSArray arrayWithObjects:@"Alice", nil, @"Eileen", @"Sepia Alice", nil];

for(NSString *string in array) {
  NSLog(@"%@", string);
}
```

The output on the console will be following:

```
Alice
```

!?!?

But wait, there's more. Print the number of elements in that array and you get:

```Objc
NSLog(@"%d", (unsigned)array.count);
```

```
1
```

`nil`, being the terminator to variadic functions, will cause the input to stop being considered the moment `nil` is found.

What happens if you want to have an array that can contain strings AND null values? Can this be done at all?

# Introducing NSNull

If `nil` is the essence of nothingness itself, `NSNull` is a *representation* of nothingness. `NSNull` contains a method `null` whose only purpose is to give you a singleton to a representation, or placeholder, of nothingness.

The Cocoa Framework is a highly dynamic thing. Because of the way `nil` behaves, particularly in Objective-C, there are many times when its pure purpose is not wanted. And so, we can answer our question:

Can we have an array of objects that also has null values?

And the answer is, yes!

And no.

`NSNull` is an actual Foundation object. It inherits from `NSObject` like anything else in the framework. In Objective-C, you cannot have an array with *pure* nil or pure nothingness. What you need to do is to replace actual nothingness with something Foundation can understand, and that's what we use `NSNull` for. You can think of `NSNull` as a *wrapper* around `nil`, but I prefer to think of it as a dummy that represents nothingness and is of the same family of anything else that comes from `NSObject`.

The last array we wrote above can therefore be rewritten like this:

```Objc
  NSArray<NSString *> *array = [NSArray arrayWithObjects:@"Alice", [NSNull null], @"Eileen", @"Sepia Alice", nil];
```

When we try to print it, we will get a more expected output:

```
Alice
<null>
Eileen
Sepia Alice
```

And if you print its `count`, you will get `4`.

Objective-C knows NSNull or something entirely different to `nil`. When you need actual nothingness, use `nil`. When you need to operate or use that nothingness, use `NSNull`.

## NSNull and Swift

In Swift, we are spared from this entire discussion, when it comes to arrays, because optionals are a data type thought around the need for nullability. If an array takes an optional, it can take `nil`, and your array will always work as you expect it to work.

A Swift optional, with the `?` marks all over the place, is really just syntactic sugar for the `Optional<T>` type, which as an enum with two cases: `.none` and `.some(T)`. Therefore, an array of optionals does not really have true nullability either, but we can say you have indirect access to nullability.

```swift
let dolls: [String?] = ["Alice", nil, "Eileen", "Sepia Alice"]

for doll in dolls {
  print(doll)
}
```

This will print:

```
Optional("Alice")
nil
Optional("Eileen")
Optional("Sepia Alice")
```

## nil-null-mess and dictionaries

Up until now, we have talked about nullability and arrays, but it's more interesting to see them when it comes to dictionaries. Nullability and dictionaries have been a point of pain for old projects for many years. If you don't understand nullability in Objective-C and Swift, you can find yourself struggling with extremely bizarre bugs.

### Objective-C Dictionaries

To explore these fun examples, we are going to parse a JSON file into a dictionary of type [`NSJSONSerialization`](https://developer.apple.com/documentation/foundation/nsjsonserialization). This would return us a dictionary of type `NSDictionary<NSString*, NSArray<NSString *> *>`. This may seem overkill, but parsing JSON the old way is the best way to describe the problems I have found in the real world and why being aware of the different "nullability" kinds is important.

```json
{
  "Pullip": [
    "Classical Alice",
    "Eileen",
    "Alice Sepia"
  ],
  "Myou": [
    "Delia",
    "Matcha"
  ],
  "HarmoniaBloom": null
}
```

In case you are following along, this is the code that loads that file (`dolls.json`) into a dictionary:

```objc
NSURL *file = [[NSBundle mainBundle] URLForResource:@"dolls" withExtension:@"json"];
NSData *data = [NSData dataWithContentsOfURL:file];
NSDictionary *dictionary = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
```

Now we are going to explore a slightly tricky situation. Now consider the following questions:

What data does `dictionary[@"Nendoroid"]` have?

What data does `dictionary[@"HarmoniaBloom"]` have?

Take a few minutes and answer in your head before continuing.

```Objc
NSLog(@"%@", dictionary[@"Nendoroid"]);
NSLog(@"%@", dictionary[@"HarmoniaBloom"]);
```

```
(null)
<null>
```

If you have read up until now, you may deduce that `null` is printed with different parenthesis or `<>` depending on whether it is `nil` or `NSNull`. But which is which?

The console will print `(null)` when printing *real* nothingness, and `<null>` when printing an NSNull.

In other words ` dictionary[@"Nendoroid"]` is `nil`, whereas `dictionary[@"HarmoniaBloom"]` is `NSNull`.

In short, in Objective-C, when a dictionary does not have a given key, it will point to `nil`. When the dictionary does the have the key, but said key is null, it will be `NSNull`. It makes complete sense if you think about it. Earlier we said `NSNull` is a placeholder for actual nothingness, and if you want a dictionary that can have nothingness, you need to use a placeholder for nothingness.

This has been the source of many weird crashes for the following reason: In Objective-C, you can send messages to `nil`, but when you are sending messages to a real object, the object needs to implement the method you want to call - in other words, you want the object to *respond to a selector*. If you send a message to an object and the object does not implement, you will crash with a message similar to "`unrecognized selector sent to instance 0xb4df00d`".

To clarify the following point, we will try to print the contents of the `Nendoroid` key, and the dolls in the `HarmoniaBloom` key, using `for in`. `for in` is the quick iteration operator. Behind the scenes, `for-in` will call the `countByEnumeratingWithState:objects:count:` method in arrays. If arrays didn't implement this method, `for-in` would crash your program instantly.

Trying to print the `Nendoroid` key won't print anything. The key does not exist in the dictionary at all:

```Objc
for(NSString *doll in dictionary[@"Nendoroid"]) {
  NSLog(@"%@", doll);
}
```

This will print nothing, and the program will move on.

On the other hand, trying to print the contents of the `HarmoniaBloom` array will have unexpected consequences:

```Objc
for(NSString *doll in dictionary[@"HarmoniaBloom"]) {
  NSLog(@"%@", doll);
}
```

```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[NSNull countByEnumeratingWithState:objects:count:]: unrecognized selector sent to instance 0x7fff8002ef10'
```

The console tells us what's wrong, and if you read the discussion above, you know what's wrong: Since accessing a non-existing key in a dictionary returns `nil`, and messages can be sent to `nil`, the program can continue executing without an issue.

On the other hand, when we access an existing key *whose value is NSNull*, the program will crash, because `NSNull` is an object like any other, and it expects to receive selector calls that it can respond to. Failing to do so will cause your program to crash.

Therefore, when accessing dynamic dictionary data, you need to check if the object that will receive a selector is `NSNull` before you send any messages to it.

```objc
if(![dictionary[@"HarmoniaBloom"] isEqual:[NSNull null]]) {
  for(NSString *doll in dictionary[@"HarmoniaBloom"]) {
    NSLog(@"%@", doll);
  }
}
```

### Dictionaries, Nullability, and Swift

**Note**: The sample code here omits working with optionals safely unless it is necessary for the example. Always work with your optionals responsibly and avoid force-unwrapping optionals as much as possible.

Unlike arrays, the above discussion is largely relevant to Swift.

First, if you want to be as type safe as possible, you will likely want to cast the result of `JSONSerialization.jsonObject(data:options:)` to something more sensible. In this case:

```swift
let jsonString =
"""
{
  "Pullip": [
    "Classical Alice",
    "Eileen",
    "Alice Sepia"
  ],
  "Myou": [
    "Delia",
    "Matcha"
  ],
  "HarmoniaBloom": null
}
"""

let data = jsonString.data(using: .utf8)!
let dictionary = try! JSONSerialization.jsonObject(with: data, options: []) as! [String: [String]]
```

The first thing you will notice here is that the program will crash with the diagnostic:

```
Could not cast value of type 'NSNull' (0x7fff86d72b38) to 'NSArray' (0x7fff86d72430)
```

This is because the values of objects may be optionals. A better option would be:

```swift
let dictionary = try! JSONSerialization.jsonObject(with: data, options: []) as! [String: [String]?]
```

Using this, the parsing will succeed without an issue. Now try printing the values of the keys `Nendoroid` and `HarmoniaBloom`:

```swift
print(dictionary["Nendoroid"])
print(dictionary["HarmoniaBloom"])
```

This will print:

```swift
nil
Optional(nil)
```

Similar to Objective-C, using the same APIs and all, accessing a nonexisting key and a key whose value is nil yield different results. `NSNull` can be used in Swift, but when bridging the framework decided not to use it.

When it comes to Swift, you will largely use Codable to parse JSON anyway, so you may not need to concern yourself much with the different types of nothingness when parsing JSON, but do keep the differences in mind.

# Conclusion

Nullability can actually be messy when working in some (rather common situations). Keep in mind what `nil` really is and what `NSNull` represents, and you will be mostly fine. To make matters worse, Objective-C has additional nothingness, including `NULL` which happens when working with C and C++ code, and `Nil`, when working with nullability of classes. Most developers do not need to concern themselves with the last two, but it's good to know they exist.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!