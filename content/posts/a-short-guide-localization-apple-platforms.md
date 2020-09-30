---
title: "A Short Guide to Localization on Apple Platforms"
date: 2020-09-23T07:00:00-04:00
originalDate: 2020-09-21T11:30:55-04:00
publishDate: 2020-09-23T07:00:00-04:00
draft: false
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
categories:
 - development
description: "Learn the technologies used to localized your iOS, iPadOS, watchOS, tvOS, and macOS apps."
keywords:
 - swift
 - ios
 - tvos
 - ipados
 - watchos
---

Translating our apps in different languages helps us reach wider audiences of different cultures. This reach can increase our app usage considerably and offer more monetization properties.

In this short article we will mention the features Apples gives us to translate our apps to different languages, namely `NSLocalizedString` and and `stringsdict` files. You will also understand when you will want to use each, as they have different use cases and an app that takes localization seriously will use both.

# NSLocalizedString and Localizable.strings

`NSLocalizedString` allows us to do simple "mappings". One string in one language corresponds to another language. Use `NSLocalizedString` when you need to create mostly static strings or dynamic strings that do not need to deal with pluralization.

For example, if you wanted to translate the words `apples`, `oranges`, `pears`:

```
NSLocalizedString("apples", comment: "The delicious red fruit")
NSLocalizedString("oranges", comment: "")
NSLocalizedString("pears", comment: "")
```

By using `NSLocalizedString`, you are telling the system to search for these strings whenever a string is found. Specify a `comment` to provide more contexts to translators. This is important because of many issues with languages, including having synonyms in English that are two different words in other languages such as Spanish, and more.

All your NSLocalizedStrings should create in your `Localizable.strings` file. This is the file that does the actual mapping of your string to other languages.

```swift
// Localizable.strings (es)

// The delicious red fruit
"apples" = "manzanas";

"oranges" = "naranjas";

"pears" = "peras";
```

You can also create strings with dynamic parameters. For example, suppose you want your app to have a string that reads "this shop sells pears". You could probably create three different localized strings for each fruit. But what happens if your fruits come from another source such as an API? You won't be able to have a string for each fruit. So you can instead create one `this shop sells "x"` string and have `x` be the right fruit. The string is a bit more complex, but nothing out of this world.

```swift
let fruitName = //... A fruit name from another source, or maybe even a local source

String.localizedStringWithFormat(NSLocalizedString("This shop sells %@", comment: "Tells the user what kind of fruits a shop sells"), fruitName)
```

And in the `Localizable.strings` file:

```swift
// Tells the user what kind of fruits a shop sells
"This shop sells %@" = "Esta tienda vende %@";
```

There are different formatters. `%@` is to specify strings and other data. You can also use `%d` for integers and all other types you'd use in `NSLog`.

You can also have strings with multiple variables. For example, to write a string such as "Number of apples available: 3":

```swift
let fruitName = NSLocalizedString("apples", comment: "The delicious red fruit")
let availableAmount = 3

String.localizedStringWithFormat(NSLocalizedString("Number of %@ available: %d", comment: "Tells the user how many of each fruit are available in the shop"), fruitName, availableAmount)
```

`NSLocalizedString` is great for mostly static strings or strings with simple parameters, but if you need pluralization, you need to work with `stringsdict` files.

# Pluralizing Strings with Stringsdict files.

In the last example of the above section, we saw this string:

```
Number of apples available: 1
```

But what if we want to say something like "There are only 3 apples left" or "There is only one apple left"? How can we do that?

You may recall that some older software always showed strings like this:

```
There are only 1 apples available
```

Functionally, the string does its job, but in this day and age, where computers are more accessible than ever, many users may find it odd.

The naïve approach would be to put if-else checks and then decide which `NSLocalizedString` to get with it. But we don't need that.

The good news is that in all of Apple's platforms, we can make it more natural by making it read "there is only 1 apple available" or my personal favorite, even "there is only one apple available" when there is only one apple left, or "there are only 3 apples available" when there's 3 left very easily. `stringsdict` files will take care of choosing the right string for us.

There are two parts to using `stringsdict`: You will still need to use `NSLocalizedString`, and then use a `stringsdict` file which is a property list of values.

`NSLocalizedString` is our entry point for any localization needs. When a `NSLocalizedString` is found, the system will look for its right counterpart in either `Localizable.strings` or the `stringsdict` file (commonly called `Localizable.stringsdict`). If the string cannot be found in a stringsdict file, the system will look for it in Localizable.strings, and if the string is in neither, it will use the raw string within the NSLocalizedString itself.

With that discussion out of the way, we will now use a `stringsdict` file to create the following strings:

```
There is only one apple left.
There are no apples left.
There are only 3 apples left.
```

Also, we will use the XML representation of the file, because it's easier to work with.

When you create a stringsdict file for the first time, you will see the following content:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>StringKey</key>
	<dict>
		<key>NSStringLocalizedFormatKey</key>
		<string>%#@VARIABLE@</string>
		<key>VARIABLE</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string></string>
			<key>zero</key>
			<string></string>
			<key>one</key>
			<string></string>
			<key>two</key>
			<string></string>
			<key>few</key>
			<string></string>
			<key>many</key>
			<string></string>
			<key>other</key>
			<string></string>
		</dict>
	</dict>
</dict>
</plist>
```

Depending on the language we are working with, we may need to pluralize differently for zero, one, two, few and many objects. In the case of English (and Spanish), we can only care about zero, one, and many, so delete all the other keys.

Start by changing `StringKey` to `APPLES_LEFT_STRING`. This `StringKey` is that one that will be used for the `NSLocalizedString` search.

Then, for `NSStringLocalizedFormatKey`, write the following string.

```
There are %#@VARIABLE@ apples left.
```

`%#@VARIABLE@` will be matched against the numeral keys (`zero`, `one`, `many`). More on this in a second.

Then, the `NSStringFormatSpecTypeKey` can only have one value for now, which is `NSStringPluralRuleType`. It's likely `stringsdict` files will have more uses beyond pluralization in the future and Apple added this for future proofing.

For `NSStringFormatValueTypeKey`, write `u`. This specifies our variable `%#@VARIABLE@` is an unsigned integer.

Then, for `zero`, `one`, and `many`, write the following:

```
<key>zero</key>
<string>There are no apples left.</string>
<key>one</key>
<string>There is one apple left.</string>
<key>many</key>
<string>There are %#@VARIABLE@ apples left.</string>
```

So, when our `%#@VARIABLE@` is 0, our string will be `There are no apples left.`, and so on.

Your final `stringsdict` file should look like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>StringKey</key>
	<dict>
		<key>NSStringLocalizedFormatKey</key>
		<string>There are %#@VARIABLE@ apples left.</string>
		<key>VARIABLE</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string>u</string>
			<key>zero</key>
			<string>There are no apples left.</string>
			<key>one</key>
			<string>There is one apple left.</string>
			<key>many</key>
			<string>There are %#@VARIABLE@ apples left.</string>
		</dict>
	</dict>
</dict>
</plist>
```

We are not done yet, we want to use this string, we need to call it and wrap it in `NSLocalizedString`:

```swift
let applesLeft = 1
String.localizedStringWithFormat(NSLocalizedString("APPLES_LEFT_STRING", comment: "Tells the user how many Apples are left"), applesLeft)
```

We do not need to specify formats for the variables in this case, as they have already been specified in the `stringsdict` file.

Now when you run that code, the string will be formatted differently depending on the value of `applesLeft`:

```
let applesLeft = 0 // There are no apples left.
let applesLeft = 1 // There is one apple left.
let applesLeft = 2 // There are 2 apples left.
let applesLeft = 49 // There are 49 apples left.
```

And for the record, you can specify strings that take more than one value, so if you needed to say "There are 6 apples spread throughout 2 boxes", you can solve that creating another variable in your stringsdict. The following is a stringsdict I use to say "Read 20 chapters out of 35":

```swift
<dict>
		<key>NSStringLocalizedFormatKey</key>
		<string>%#@VOLUMES@ %#@TOTAL_VOLUMES@</string>
		<key>VOLUMES</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string>u</string>
			<key>other</key>
			<string>read %u</string>
			<key>zero</key>
			<string>read nothing</string>
			<key>one</key>
			<string>read one</string>
		</dict>
		<key>TOTAL_VOLUMES</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string>u</string>
			<key>other</key>
			<string>out of %u volumes</string>
			<key>zero</key>
			<string></string>
			<key>one</key>
			<string>out of one volume</string>
		</dict>
	</dict>
	<key>EPISODES_BEHIND_TEXT</key>
	<dict>
		<key>NSStringLocalizedFormatKey</key>
		<string>%#@EPISODE_COUNT@</string>
		<key>EPISODE_COUNT</key>
		<dict>
			<key>NSStringFormatSpecTypeKey</key>
			<string>NSStringPluralRuleType</string>
			<key>NSStringFormatValueTypeKey</key>
			<string>u</string>
			<key>zero</key>
			<string>No episodes behind</string>
			<key>one</key>
			<string>one episode behind</string>
			<key>other</key>
			<string>%u episodes behind</string>
		</dict>
	</dict>
```

# Conclusion

Localization is important, even more in modern computers, even more so. Apple provides developers with tools to make both "direct" translations easier and to make pluralization make sense in the context of any language.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.