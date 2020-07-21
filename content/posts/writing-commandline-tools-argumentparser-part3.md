---
title: "Writing Command Line Tools in Swift Using ArgumentParser, Part 3: Subcommands"
date: 2020-04-01T07:00:00-04:00
draft: false
originalDate: 2020-03-25T18:47:17-04:00
publishDate: 2020-04-01T07:00:00-04:00
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
description: "Organizing ArgumentParser tools in different subcommands."
keywords:
 - swift
 - programming
 - apple
 - ArgumentParser
---

We have been having a lot of fun with ArgumentParser in the last two weeks, and the fun is not about to end any time soon. We have explored how we can [build basic commands with the basic building blocks of the framework](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part1/), and how we can perform [advanced validation and error handling](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part2/). This week, we will something very useful: Subcommands.

# Subcommands

If you have used `git` directly from the command line before, you have used subcommands before.

Consider the following example:

```
git add .
```

In `ArgumentParser` talk, `git` would be a command - something that inherits directly from `ParsableCommand`. What's with the `add`? An `@Argument`? You can actually probably find a way to implement this with an `@Argument`, but there is actually an easier way to implement something like this.

By using subcommands, we can create commands that wrap different subcommands. In the above example, `git` is the main command, and `add` is a subcommand. It can be a good idea to separate your command line tool into different subcommands as it grows. The beautiful thing about `ArgumentParser` is that it provides many features to make this separation easier, in the Swiftiest way possible.

We will explore these features by creating sub commands for our `CharacterCount` tool: One to count characters for a string we passed directly; One to count the characters from a local file; and finally, one to count the characters from a remote URL. 

## The ParsableArguments Protocol

Implement this protocol when you need to create properties that will be shared across your subcommands. Types that conform to this protocol can parse arguments handed through the command line, but they cannot `run` on their own.

```
struct CharacterCount: ParsableCommand {
  
  enum CountingConfiguration: String, CaseIterable {
    case all
    case uppercaseOnly
    case lowercaseOnly
  }
  
  struct Options: ParsableArguments {
    @Flag(default: CountingConfiguration.all, help: "The kind of characters to count") var countingConfig: CountingConfiguration
    
    @Flag(help: "If set, ignores whitespace characters") var ignoringWhitespace: Bool
    
    @Option(default: 1, help: "Multiplies the end result by the specified number") var multiplier: Int
  }
}
```

So far nothing too fancy. We have created an `Options` struct that conforms to `ParsableCommand` and we have added a few properties there.

## Creating Subcommands

To actually create a subcommand, we need to define them just the same way you would define a parent command, conforming to `ParsableCommand` and all. Then, you need to tell your parent command that it contains the subcommands with their name.

You can begin separating your code in separate files for organization purposes. And then you can define the subcommands within extensions of the parent command.

```swift
extension CharacterCount {
  struct DirectString: ParsableCommand {
    @Argument(help: "The string to count the characters of") var string: String
    
    func run() {
      print(string.count)
    }
  }
}
```

This is the basic implementation. We will implement the options in a bit.

Next, to actually create the relationship that your parent command has subcommands, we need to create a `CommandConfiguration` property where we can specify each subcommand that belongs to it. We will explore `CommandConfiguration` in depth in a later article. For now, you can use it like this to define your subcommands. Add the following property to your parent command:

```swift
static let configuration = CommandConfiguration(subcommands: [DirectString.self])
```

With all that done, we can now call our subcommand:

```
./MyCommandLineTool direct-string "Alice"
5
```

The `direct-string` name was generated for you for free. The help page for the parent command now has a `SUBCOMMANDS` section:

```
./MyCommandLineTool                     
USAGE: character-count <subcommand>

OPTIONS:
  -h, --help              Show help information.

SUBCOMMANDS:
  direct-string   
```

`CommandConfiguration` can also take a `defaultCommand` to run, which you can use when your command is run without specifying a subcommand.

### Propagating Options with @OptionGroup

Introducing now a new property wrapper to your `ArgumentParser` toolbox, we have `@OptionGroup`. This property wrapper allows us to receive the arguments defined in a `ParsableArguments` type.

To use, simply add a line like the following in a subcommand:

```swift
@OptionGroup() var parentOptions: Options
```

Where `Options` is the type we defined in the parent.

We can now use them in our `DirectString` command. The full implementation for this subcommand looks like this:

```swift
extension CharacterCount {
  struct DirectString: ParsableCommand {
    @Argument(help: "The string to count the characters of") var string: String
    
    @OptionGroup() var parentOptions: Options
    
    func run() {
      let whiteSpacechars = string.filter { $0 == " " }.count
      let alwaysSubtract = parentOptions.ignoringWhitespace ? whiteSpacechars : 0
      let mult = parentOptions.multiplier
      
      if parentOptions.countingConfig == .all {
        print((string.count - alwaysSubtract) * mult)
      }
      
      if parentOptions.countingConfig == .uppercaseOnly {
        let count = string.filter { $0.isUppercase }.count
        print((count - alwaysSubtract) * mult)
      }
      
      if parentOptions.countingConfig == .lowercaseOnly {
        let count = string.filter { $0.isLowercase }.count
        print((count - alwaysSubtract) * mult)
      }
    }
  }
}
```

You can now use all the options defined in the parent, and all subcommands that belong to the parent belong can use the options in the same way. As a reference, in our program you'd use the options as:

```
./MyCommandLineTool direct-string "Pullip Classical Alice"                      
22

./MyCommandLineTool direct-string "Pullip Classical Alice" --ignoring-whitespace
20

./MyCommandLineTool direct-string "Pullip Classical Alice" --ignoring-whitespace --multiplier 3
60

```

## Implementing The Other Commands

You have now learned how to implement subcommands and how to use `OptionGroup`, so you have all the tools you need to implement the other subcommands. If you don't feel like doing so, I will leave their implementations below:

```swift
extension CharacterCount {
  struct LocalFile: ParsableCommand {
    @Argument(help: "A path to a local file to count the characters of") var localFile: String
    
    @OptionGroup() var parentOptions: Options
    
    func run() {
      do {
        let string = try String(contentsOfFile: localFile)
        processString(string: string, options: parentOptions)
      } catch {
        print("Unable to open local file")
      }
    }
  }
}

extension CharacterCount {
  struct RemoteFile: ParsableCommand {
    @Argument(help: "The URL of the remote file to count the characters of", transform: { URL(string: $0)! }) var remoteFile: URL
    
    @OptionGroup() var parentOptions: Options
    
    func run() {
      do {
        let string = try String(contentsOf: remoteFile)
        processString(string: string, options: parentOptions)
      } catch {
        print("Unable to open local file")
      }
    }
  }
}

func processString(string: String, options: CharacterCount.Options) {
  let whiteSpacechars = string.filter { $0 == " " }.count
  let alwaysSubstract = options.ignoringWhitespace ? whiteSpacechars : 0
  let mult = options.multiplier
  
  if options.countingConfig == .all {
    print((string.count - alwaysSubstract) * mult)
  }
  
  if options.countingConfig == .uppercaseOnly {
    let count = string.filter { $0.isUppercase }.count
    print((count - alwaysSubstract) * mult)
  }
  
  if options.countingConfig == .lowercaseOnly {
    let count = string.filter { $0.isLowercase }.count
    print((count - alwaysSubstract) * mult)
  }
}
```

And don't forget to add them as subcommands in the parent command:

```swift
  static let configuration = CommandConfiguration(
    subcommands: [
      DirectString.self,
      RemoteFile.self,
      LocalFile.self
    ]
  )
```

# Conclusion

Separating your command line tool into subcommands is very easy to do thanks to `ArgumentParser`'s parser features. You can configure children commands very easily in a parent command's configuration, and, if your subcommands take the same options, flags, and arguments, you can declare them in a type conforming to `ParsableArgument`, so all subcommands that need them can simply use the `@OptionGroup` property wrapper to access them.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.