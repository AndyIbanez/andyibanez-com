---
title: "Writing Command Line Tools in Swift Using ArgumentParser, Part 4: Customizing Help"
date: 2020-04-08T07:00:00-04:00
draft: false
originalDate: 2020-04-03T12:27:36-04:00
publishDate: 2020-04-08T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ArgumentParser
categories:
 - development
description: "Learn how to customize the help pages of your ArgumentParser command to provide better documentation."
keywords:
 - swift
 - programming
 - apple
 - ArgumentParser
---

Writing Command Line Tools in Swift Using ArgumentParser, Part 4: Customizing Help

In the past few weeks, we have explored how to use `ArgumentParser` and many of its features. It's great that `ArgumentParser` provides a lot of functionality for free, but it wouldn't make sense to build great tools that users can't figure out how to use. This week is all about that.

We saw how `ArgumentParser` can build a lot of documentation for free, but we can actually do more. This week, we will explore how we can improve the documentation generated for our command line tools.



## Customizing Help For Options, Arguments, and Flags

`@Argument`, `@Option`, and `@Flag` can take a `help` property which we can use to describe the parameter and how to use it. But not only can this take a string, it can also take an `ArgumentHelp` object (despite it being called `ARGUMENTHelp`, it can be used in all the property wrappers).

```
  @Argument(help:
    ArgumentHelp(
      "The string parameter will be counted against the specified character sets",
      discussion: "This obligatory parameter will be used to count the characters of.",
      valueName: "theString",
      shouldDisplay: true)) var string: String
```

```
andyibanez@Andys-iMac Debug % ./MyCommandLineTool --help
USAGE: character-count <theString> [--whitespace] [--numbers] [--vowels]

ARGUMENTS:
  <theString>             The string parameter will be counted against the
                          specified character sets 
        This obligatory parameter will be used to count the characters of.

OPTIONS:
  --whitespace/--numbers   
  --vowels                 
  -h, --help              Show help information.
```

You can use this object to customize every aspect of the parameter's help. The `discussion` is a short description next to the parameter name; the `valueName` is a customized name you can use if you don't want the framework to generate one for you automatically; Finally, `shouldDisplay` is a boolean that triggers whether the parameter should be shown or not. This is handy when you want to hide certain properties.

# Customizing a Command's Help Via CommandConfiguration

We explored how to use `CommandConfiguration` when we talked about [subcommands](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part3/), this little object can do much more, including customize our command's entire help page. In other words, what your users see when they run your tool with the `-h` or `--help` flags.

```swift
  static let configuration = CommandConfiguration(
    commandName: "CharacterCounter",
    abstract: "Allows you to count the number of characters in a string",
    discussion: "A string is a made up of multiple characters. A character can be human-readable or a control character. When counting characters, you may need to know if you want to consider control characters or not, as the results may vary.")
```

The `CommandName` is the name we want our tool to have, the name we want our users to invoke when they want to use our command line tool. This is helpful if you do not necessarily want the command name to be the executable name.

The `abstract` is a short description of what the command line tools. It should give your users a quick overview of what your tool does.

The `discussion` can be a longer description. You can format it using Swift heredoc-style strings. Your command line tool can give more information and context by using this property.

When your user runs your tool with `-h`, they will see this:

```
andyibanez@Andys-iMac Debug % ./MyCommandLineTool -h
OVERVIEW: Allows you to count the number of characters in a string

A string is a made up of multiple characters. A character can be human-readable
or a control character. When counting characters, you may need to know if you
want to consider control characters or not, as the results may vary.

USAGE: CharacterCounter <string>

ARGUMENTS:
  <string>                String to count 

OPTIONS:
  -h, --help              Show help information.
```

And that's it! Configuring our help pages is really use, and we can write user-facing documentation with a few lines of code.

# Conclusion

Your command line tool should provide as much help as possible for your users. ArgumentParser makes it very easy to write them, by providing `ArgumentHelp` and `CommandConfiguration`.

