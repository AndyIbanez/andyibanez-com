---
title: "The NSDateInterval Object"
date: 2021-05-05T07:00:00-04:00
originalDate: 2021-05-02T23:40:41-04:00
publishDate: 2021-05-05T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - apple
 - programming
 - foundation
 - nsdate
categories:
 - development
keywords:
 - swift
 - apple
 - programming
 - foundation
 - nsdate
---

The NSDateInterval Object

Somehow, this shiny new object, which was actually introduced in iOS 10, flew past my radar. Today I want to take a few minutes to talk about the `NSDateInterval` object. This object allows us to quickly calculate the time interval (represented as a `NSTimeInterval`) between dates, it allows us to check if two dates overlap, and it allows us to check if a given date is within a certain interval.

# The NSDateInterval Class

This small object is made of a handful of property and functions.

To create a `NSDateInterval`, you can provide either a closed range with the start and end date, or you can provide the start date with a duration. In the example below we will use two dates, seven days apart.

```swift
let now = Date()

let components = DateComponents(day: 7)
if let sevenDaysAhead = Calendar.current.date(byAdding: components, to: now) {
    let interval = DateInterval(start: now, end: sevenDaysAhead)
}
```

Our `internal` variable now has a couple of handful properties. You can get the `startDate` and `endDate`, but perhaps most interesting the `duration`, which contains the number of seconds between both dates.

Having the duration can be useful on its own, but what's even more useful is that we can now compare and check if said date intervals intersect, or if a date is part of a certain date interval.

## Comparing Date Intervals

To compare two `DateIntervals`, simply call `compare(_)` on one of them and pass it the second interval. This operation will return a [`ComparisonResult`](https://developer.apple.com/documentation/foundation/comparisonresult) which will let you know which one is "bigger" than the other.

```swift
let now = Date()

let components = DateComponents(day: 7)
if let sevenDaysAhead = Calendar.current.date(byAdding: components, to: now) {
    let interval = DateInterval(start: now, end: sevenDaysAhead)
    
    // Let's create a differemt but similar interval, but eight months ago
    let eightMonthsAgoComponents = DateComponents(month: -8)
    if let eightMonthsAgo = Calendar.current.date(byAdding: eightMonthsAgoComponents, to: now),
       let eightMonthsAgoPlus7Days = Calendar.current.date(byAdding: DateComponents(day: 7), to: eightMonthsAgo){
        let interval8MonthsAgo = DateInterval(start: eightMonthsAgo, end: eightMonthsAgoPlus7Days)
        
        let comparison = interval.compare(interval8MonthsAgo)
        switch comparison {
        case .orderedAscending: print("orderedAscending")
        case .orderedDescending: print("orderedDescending")
        case .orderedSame: print("orderedSame")
        }
    }
}
```

This is a bit of a mouthful, but essentially what it does is:

1. Create an interval between the current day and the date seven days in the future.
2. Create a date 8 months in the past, create a date 7 days after that day in the past, and create an interval with the two.
3. Compare them.

The result will be `.orderedDescending`, since the left side of the operation (`interval`) is more recent than `interval8MonthsAgo`. When comparing, the framework takes into account the `startDate` and the `duration` if necessary. The [documentation](https://developer.apple.com/documentation/foundation/nsdateinterval/1641636-compare) on comparing intervals has more info.

## Equality of DateIntervals

We can check if two intervals are equal by using the `==` operator (the documentation incorrectly states that there is a [`isEqual`](https://developer.apple.com/documentation/foundation/nsdateinterval/1641650-isequal) function, but I wasn't able to access it.

Two `DateInterval`s are considered equal when their `startDate` and `duration` properties are the same.

```swift
let equalIntervals = interval == interval8MonthsAgo // false
```

The take away from the last two sections is that equality and comparison takes into account the `startDate` and `duration` properties in order to do their calculations. If you need to see if two date ranges are the same duration, but they don't overlap or have any relation with each other, use `Calendar`'s `Calendar.current.dateComponents((_from:to:)` method instead.

```swift
let difference = Calendar.current.dateComponents([.day], from: now, to: eightMonthsAgo) // -242 days ago (feel free to use abs() for the absolute value
```

## Checking Interval Intersections

You can check if two `DateInterval`s intersect by calling the `intersects(_)` method in one of them.

```swift
let now = Date()

if let sevenDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 7), to: now),
   let sixDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 6), to: now),
   let eightDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 8), to: now),
   let ninetDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 9), to: eightDaysAhead){
    let sevenDayInterval = DateInterval(start: now, end: sevenDaysAhead)
    let sixDayInterval = DateInterval(start: now, end: sixDaysAhead)
    let eightDaysInterval = DateInterval(start: now, end: eightDaysAhead)
    let sevenAndSixIntersect = sevenDayInterval.intersects(sixDayInterval) // true
    let sevenAndEightIntersect = sevenDayInterval.intersects(eightDaysInterval) // true
    
    let farAwayInterval = DateInterval(start: eightDaysAhead, end: ninetDaysAhead)
    let sevenDaysIntervalIntersectsFarAwayInterval = sevenDayInterval.intersects(farAwayInterval) // false
}
```

And, you can also find the interval at which two date intervals intersect, by calling `intersection(with:)` on either one.

```swift
import Foundation

let now = Date()

if let sevenDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 7), to: now),
   let sixDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 6), to: now),
   let eightDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 8), to: now),
   let ninetDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 9), to: eightDaysAhead){
    let sevenDayInterval = DateInterval(start: now, end: sevenDaysAhead)
    let eightDaysInterval = DateInterval(start: now, end: eightDaysAhead)
    
    if let intersectionInterval = sevenDayInterval.intersection(with: eightDaysInterval) {
        print("The intervals intersect starting on \(intersectionInterval.start) and ending on \(intersectionInterval.end)")
    }
}
```

## Checking if a date exists within an interval.

Finally, the last useful thing we can do is to check if a single date fits in an interval. For this, simply call `contains(_)`.

```swift
let now = Date()

if let sevenDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 7), to: now),
   let sixDaysAhead = Calendar.current.date(byAdding: DateComponents(day: 6), to: now),
   let sevenDaysAndASecondAhead = Calendar.current.date(byAdding: DateComponents(day: 7, second: 1), to: now) {
    
    let intervalSevenDaysAhead = DateInterval(start: now, end: sevenDaysAhead)
    
    let containsSixDaysAhead = intervalSevenDaysAhead.contains(sixDaysAhead)
    let containsSevenDaysAndASecondAhead = intervalSevenDaysAhead.contains(sevenDaysAndASecondAhead)
}
```

# Conclusion

If there is one thing Apple likes to do, is to make dealing with dates a simple affair for us. All their date-related APIs are packed with functionality to make calculation and manipulation of time-related data really easily. 5 years ago, Apple added `NSDateInterval` to their collection of date tools, and it's amazing how simple it is to work date intervals with it.



