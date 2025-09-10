---
title: "Understanding Basic Data Structures in Swift: Sets"
date: 2021-01-20T07:00:00-04:00
originalDate: 2021-01-17T15:35:00-04:00
publishDate: 2021-01-20T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - apple
 - sets
 - data structures
 - data
 - structures
 - foundation
categories:
 - development
description: "Learn what sets are and how to use them in Swift."
keywords:
 - swift
 - apple
 - sets
 - data structures
 - data
 - structures
 - foundation
---

I'm introducing a short small series in which we will talk about basic data structures in Swift. My goal is not to show how they are implemented internally, but rather to show when they can be useful.

In truth, unless you have studied Computer Science to some capacity, chances are you are missing on a lot of powerful existing data structures that can help you write better code. I have been studying iOS development for a long time with many resources, and none of the resources ever dive into useful data structures, such as *sets*. These sources tend to focus on arrays and dictionaries only (as the focus is iOS development, and not necessarily computer science), not teaching other structures that are actually really useful in the iOS Development world. I have never seen an iOS dev resource that covered these structures as deeply as my computer systems engineering courses did.

Many computer science classes introduce data structures by teaching students the theory and making them implement them. We are not going to do be doing that here (unless there's demand for that, for a future article). Instead, this series will introduce a new structure and show you the native implementation in either Foundation or the Swift standard library.

Some of these articles may contain a bit of mathematical discussion. I do not want that to be a turn off to read them. I can guarantee you, as someone who has never had a good mathematical foundation, that understanding these concepts is both easy and important to grow your career.

Most of the examples here use numbers, but keep in mind that you can use any other object with sets as long as it complies with the `Equatable` protocol. Many types, like `String`s, are already Equatable so you can use those in sets right out of the box.

Without further ado, let's get into it.

# Introduction to Sets.

[Stanford University](https://plato.stanford.edu/entries/set-theory/) defines set theory as:

> Set theory is the mathematical theory of well-determined collections, called sets, of objects that are called members, or elements, of the set. 

In more humane words, a Set is a collection of some sort of object. These objects can have sub-groupings, or *sub sets*, that allow us to further constraint the objects a set can contain.

You may remember from your elementary school days, that numbers can be grouped in multiple sets:

**Natural numbers**: Whole numbers from 1 upwards.
**Integers**: Whole numbers, which include both negative numbers and the number 0.
**Rational Numbers**: Numbers that result from dividing one integer by another, except division by zero. Essentially, fractions.

These are just some of the sets you have seen in school, but what matters here is that *Numbers* themselves are a set, and you can subset Numbers into many different categories. Integers are a *subset* of numbers, and Natural numbers are a subset of integers. We can further find more sets that were defined way before our time, and we can even create our own sets too, like numbers that can be divisible by 2 with a remainder of 0, and more.

## Sets and Arrays

Many times, I see programmers using Arrays when a Set may suffice. What you may be wondering at this point is what is the relationship of a set with an array? How can one replace the other?

In computer science, sets have an additional property, and that is that all its members are *unique*. Many libraries and frameworks offer you Sets with a an existing definition of Unique for specific data types. For example, in Swift, the definition of uniqueness for a set or integers is that each number must be different than any other existing number, but you can create your own definition of uniqueness for your custom types.

In contrast, an array can contain multiple elements, including repeating ones.

Many times I see programmers wanting to keep an array of unique elements, and swiftly making up ways to clean their arrays of duplicate elements. In Swift, if you need an array of unique elements, you can quickly convert between Sets and Arrays and vice-versa with an initializer. Both sets and arrays are *Collections* in Swift, and interoperating between them is easy.

Consider the following example:

```swift
let array = [1, 1, 2, 3, 4]
let set = Set(array)
```

We declare an array called `array` with a list of elements, and another set with the same list of elements.

If you count the number of elements in each:

```
print("Array count: \(array.count) - Set count: \(set.count)") // 
```

You will find that `array` has more elements than `set`.
> Array count: 5 - Set count: 4

We have magically converted an array into a set by just passing the array to the set. The set has done its magic and it has removed all the duplicate numbers for us.

## Sets and Ordering

Using sets has another implication though: They are not ordered by default, and there is no order to be found in the definition of a set. Some implementations in other libraries and languages may guarantee an order, but conceptually you should never rely on that.

If you `print(set)` multiple times, you will see the output is different every time.

Luckily, we can get a sorted version of the set easily by just calling `sorted()` or `sorted(by:)`.

```swift
print(set.sorted()) // [1, 2, 3, 4]
print(set.sorted(by: >)) // [4, 3, 2, 1]
```

Do note that, if you need to keep the set ordered to begin with - every insertion inserts the item in the right place -, you will need to use something different. Foundation provides [NSOrderedSet](https://developer.apple.com/documentation/foundation/nsorderedset) for this. Because it's an old Foundation API, it does not support Swift generics, so using it requires a bit of involved code.

## Set Operations

Now let's talk about common set operations you can do. All the set operations I am going to talk to about take two sets and return another set. Keep these in mind as you will likely find moments where it's useful to know them in your code.

### Intersection

The **intersection** operation returns a new set of elements that exist in both sets.

```swift
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4, 5, 6]

let intersection = set1.intersection(set2)
print(intersection) // [3, 4]
```

In the above example, `intersection` only contains `3` and `4`, as those are the only items that exist in both sets. Do note that there is no guaranteed order in the resulting set, so you can expect a different order every time.

### Difference

Given `set1` and `set2`, the difference of two sets are the elements that are unique to each set. It is essentially the opposite of intersection.

In Swift, `symmetricDifference` will return a new set.

```
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4, 5, 6]

let difference = set1.symmetricDifference(set2)
print(difference) // [6, 5, 2, 1]
```

Swift also has the `formSymmetricDifference` method for sets. This call will remove the elements that exist in both sets, and it will append the resulting elements from the second set to the first one:

```swift
var set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4, 5, 6]

set1.formSymmetricDifference(set2)
print(set1) // [6, 5, 2, 1]
```

Do note that because this mutates a set, the "target" set has to be declared as `var`.

### isDisjoint

Given a `set1` and `set2`, `set1` is *disjoint* with `set2` if `set2` contains only elements that are not part of `set1`.

```swift
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4, 5, 6]
let set3: Set = [5, 6]

set1.isDisjoint(with: set2) // false
set1.isDisjoint(with: set3) // true
```

### Superset Checking

You can check if `set1` is a *superset* - in other words, that `set1` contains all the elements - of `set2`.

We say `set1` is a superset of `set2` if every member of `set2` also belongs to `set1`.

```swift
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4]
let set3: Set = [4, 6]

set1.isSuperset(of: set2) // true
set1.isSuperset(of: set3) // false
```

We can also check if `set1` is a *strict superset* of `set2`.

`set1` is a strict superset of `set2` when all the elements in `set2` exist in `set1`, and when `set1` contains at least one element that is not part of `set2`.

```swift
let set1 = Set([1, 2, 3, 4])
let set2 = Set([3, 4])

set1.isStrictSuperset(of: set2) // true
```

Because of this, a set is a *superset* of itself, but a set will never be a *strict superset* of itself.

### Subset Checking

Just like we can check when a set is a superset or strict superset of another set, we can do the same with subsets.

`set2` is a *subset* of `set1` when all the elements of `set2` exist on `set1`.

```swift
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4]

set2.isSubset(of: set1) // true
```

`set2` is a *strict subset* of `set1` when every member of `set2` is also a member of `set1` and `set1` contains at least one element that is not part of `set2`.

```swift
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4]

set2.isStrictSubset(of: set1) // true
```

### Subtraction

From `set1`, you can remove all the elements that exist in `set2`.

There are two ways of doing this: The first method returns a new set containing the elements of `set1` that were removed because they were in `set2`.

```swift
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4]

let subtraction = set1.subtracting(set2) // [1, 2]
```

The other one takes a mutating set, and the subtraction occurs on it.

```swift
var set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4]

set1.subtract(set2) // [1, 2]
```

### Union

The union between `set1` and `set2` contains all the elements of both sets, with the duplicate ones removed.

```swift
let set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4, 5, 6]

let union = set1.union(set2) // [1, 2, 3, 4, 5, 6]
```

`formUnion` does the same, but operating on a mutable set instead.

```swift
var set1: Set = [1, 2, 3, 4]
let set2: Set = [3, 4, 5, 6]

set1.formUnion(set2) // [1, 2, 3, 4, 5, 6]
```

### Inserting and Updating

You can insert new elements to mutating sets. For this, we have the `insert` and `update` methods.

They work very similarly, with the difference living in the return object.

```swift
var set1: Set = [1, 2, 3]
let set2: Set = [3, 4, 5, 6]

set1.update(with: 4) // returns 4
set1.insert(7) // returns (true, 7)
```

`update` returns the object that was just inserted, or `nil` if the object could not be inserted because it already exists.

`insert` returns a boolean indicating if the member was inserted, and the value of the `last` member that was equal to the inserted one. This is important, because with more complicated member types, you may have logic to consider equality for sets, but ultimately your objects may have differences.

## High-order Functions

Because Sets in Swift are implemented as Collections, you can use `filter`, `map`, `reduce` and other high order functions on them.

```swift
let set: Set = [1, 2, 3, 4]
let set2: Set = [3, 4, 5, 6]

let filtered = set.filter { $0 % 2 == 0 } // [2, 4]

```

## APIs based on Sets

Finally, I want to talk about other APIs that are implemented as sets and are really useful. In particular, I want to talk about Foundation's `Character set`s.

The sets you have seen until now can be really useful and I fully expect to see such code in the real world, but character sets are very useful, especially when dealing with strings.

Introduced in iOS 7, [CharacterSet](https://developer.apple.com/documentation/foundation/characterset) gives us access to predefined character sets and allows us to create our own. These sets can help us create some string validations easier.

```swift
let letters = CharacterSet.letters

let string = "pullipalice"
let stringCharacters = CharacterSet(charactersIn: string)

stringCharacters.isSubset(of: letters) // true
```

We are checking a simple string and seeing if it has only letters. `CharacterSet` has many predefined sets for us, such as `letters`, `alphanumeric`, `decimalDigits`, and more. We can also very easily create our own character sets. In the following example, we will create a set with morse code characters and compare it against some strings.

```swift
let morseCodeCharSet = CharacterSet(arrayLiteral: ".", "-")
let string1 = "..--.-"
let string2 = "alice..-.."
let string3 = "------"

let string1Set = CharacterSet(charactersIn: string1)
let string2Set = CharacterSet(charactersIn: string2)
let string3Set = CharacterSet(charactersIn: string3)

morseCodeCharSet.isSuperset(of: string1Set) // true
morseCodeCharSet.isSuperset(of: string2Set) // false
string3Set.isSubset(of: morseCodeCharSet) // true
```

# Conclusion

Sets are a very powerful but lesser used data structure. They have al of features and they can take up the place of arrays more often than we would think. It's worth to study them, as they will help us become better developers.

