---
title: "Using NSMeasurement For Working with Dimensions and Units"
date: 2021-03-24T07:00:00-04:00
originalDate: 2021-03-19T22:49:08-04:00
publishDate: 2021-03-24T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - nsformatter
 - swift
 - programming
 - apple
 - ios
 - ipados
categories:
 - development
description: "Learn how to use NSMeasurement to work with dimensions and units."
keywords:
 - nsformatter
 - swift
 - programming
 - apple
 - ios
 - ipados
---

Software development can be an easy thing, as it can be a very complex thing. And one of those complex things is keeping in mind all the different languages, locations, and standards users may use in their daily lives. This makes working with certain information. From different date formats to entirely different measuring system, software is challenging, especially when working with anything that requires localization. The situation is just so bad that a lot of software just make assumptions about their users environment and don't let you change any settings.

For many localization needs, we can make use of [`NSFormatter`](https://www.andyibanez.com/tags/nsformatter/). When it comes to dealing with different units for any daily-life measurement - speed, weight, area, acceleration, etc -, we can make use of `NSMeasurement`. Introduced in iOS 10, this class and its friend, `NSMeasurementFormatter`, allow us to work with different units in any system, perform calculation with them, and ultimately display them to our users.

In this article, we will explore how `NSMeasurement` works, and how to pair it `NSMeasurementFormatter` so users using your app can always expect to see their preferred units in your app.

# Dimension and Units are Everywhere

If you thought these APIs were useful only for writing unit converter apps, I got some rough news for you. Units are actually used in many, many places. All the way from measuring time to the length of an object, users have apps that keep track of this data without thinking about it.

For this reason, it is important to work correctly with units, because they can show app even in shopping list apps, and even games. One particular annoying thing from my childhood - although very minor - was playing Need for Speed and only seeing the units in Imperial units when that's not what I use. It did help me to learn and understand different countries and people use different units for something I had for granted, but that's not what I wanted when I wanted my shiny new car to win races are escape the scope in Need for Speed: Most Wanted.

In the same vein, if you took a physics class in school and you studied in the United States, you may remember the fun times you had to convert between units all the time because science uses metric.

What I'm trying to tell you here is that even games can make good use of operating on generic units and presenting them to your users in the way they expect. The need for displaying the right units is basically everywhere.

## Supported Units

Enough chatter for now, and let's get into the point of the article.

The need for units is so important that `NSMeasurement` supports many of them, including ones you may have never heard of.

For the common uses, you can work with length, mass, duration, acceleration, and many more. Trying to cover them all would probably require a small book, so I will only use examples that uses dimensions we are all familiar with. If you want a more complete reference, take a look at the [`Dimension`](https://developer.apple.com/documentation/foundation/dimension) docs. This page also lists the base unit for each.

It's worth noting that the base units for dimensions appear to prefer metric units, so the base unit for length is meters; the base unit for mass is kilograms; the base unit for duration is seconds, and the base unit for acceleration is m/s^2.

## Sample Units

To create a `NSMeasurement`, you need to provide it with value and a unit.

```swift
let weight = Measurement(value: 2.0, unit: UnitMass.kilograms)
```

One of the beauties of this API is that you can create any measurement in any compatible unit and operate on them. In the example below, we will add a weight in kilograms and another one in grams:

```swift
let weight = Measurement(value: 2.0, unit: UnitMass.kilograms)
let weightInGrams = Measurement(value: 1500, unit: UnitMass.grams)

print(weight + weightInGrams)
```

The API will convert the result to the base units before showing the result. This, this will print `3.5 kg`.

And don't worry about adding incompatible types. If you try to add different dimensional units, the generic will protect you at compile time.

```swift
let weight = Measurement(value: 2.0, unit: UnitMass.kilograms)
let speed = Measurement(value: 1000, unit: UnitSpeed.kilometersPerHour)

print(weight + speed) // Won't compile
```

You can easily convert between different units by calling the `converted(to:)` method.

```swift
let weight = Measurement(value: 2.0, unit: UnitMass.kilograms)
let weightInGrams = Measurement(value: 1500, unit: UnitMass.grams)

let totalWeight = weight + weightInGrams
let totalWeightInPounds = totalWeight.converted(to: .pounds)

print(totalWeightInPounds) // prints "7.716185470643222 lb"
```

Finally, you can actually compare between two `NSMeasurement` very easily using the standard operators you know.

```swift
if weight > weightInGrams {
  print("Got more in kgs")
}
```

The amount of work this object does to help you work with units is nothing less than mind blowing.

## Using NSMeasurementFormatter For User Facing Units

Everything we did so far is great if we don't need to show anything to the user. While printing to the console does append a measurement, the right way to show users a value is by using a formatter.

The formatter will do more than just displaying the value correctly to the user. You can configure it with many more parameters.

I will force the locale to use a metric system instead of Imperial, as that makes more sense to me (my simulator is set in the USA and therefore the formatter uses Imperial units). I can do this by setting the locale of the formatter.

```swift
let formatter = MeasurementFormatter()
formatter.locale = Locale(identifier: "es_BO")
```

### Some Formatter Configs

#### unitOptions

If you set the unitOptions (`UnitOptions`), you can choose the behavior of what to do with the provided unit. If you use `.providedUnit`, the formatter will format and display the measurement with the unit you used to create it.

```swift
let weightInGrams = Measurement(value: 1500, unit: UnitMass.grams)

let formatter = MeasurementFormatter()
formatter.locale = Locale(identifier: "es_BO")
formatter.unitOptions = .providedUnit;

print(formatter.string(from: weightInGrams)) // prints "1,500 g"
```

Using `.naturalScale` will cause the value to be formatted into a "bigger" unit if possible. For example, if you have a measurement in grams, that can be represented into kilograms, the formatter will do that conversion for you.

```swift
let weightInGrams = Measurement(value: 1500, unit: UnitMass.grams)

let formatter = MeasurementFormatter()
formatter.locale = Locale(identifier: "es_BO")
formatter.unitOptions = .naturalScale

print(formatter.string(from: weightInGrams)) // prints "1,5 kg" (note that in Bolivia we use a comma to separate decimals, not thousands).
```

#### unitStyle

Setting the unitStyle (`UnitStyle`) will change how the unit is spelled out.

```swift
// 1,5 kilogramos
formatter.unitStyle = .long
// 1,5 kg
formatter.unitStyle = .medium

// 1,5kg
formatter.unitStyle = .short
```

I have willingly left my country's units there as they perfectly show why using a formatter and the right unit for users is important.

#### NumberFormatter

You can pass an entire `NSNumberFormatter` to choose how your number will be formatted. This is useful if your locale settings don't cover very specific cases or personal user preference.

A particular preference for me is that I don't use commas for decimals like we do in my country. I like using periods instead, so I can keep using my locale and just set the decimal separator by setting the numberFormatter

```swift
formatter.unitStyle = .short

formatter.numberFormatter.decimalSeparator = "."

print(formatter.string(from: weightInGrams)) // prints 1.5kg
```

# Conclusion

Once again we find ourselves talking about formatters, but this time with measurements included. Many people have different uses for measurements and despite locale expectations, users may need to work with units differently. `NSMeasurement` and `NSMeasurementFormatter` provide us with many tools to quickly work with different units, all the way from operations to displaying them, in a quick and efficient manner.

