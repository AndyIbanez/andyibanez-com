---
title: "Writing Command Line Tools in Swift Using ArgumentParser, Part 5: Tools with Asynchronous APIs"
date: 2020-04-15T07:00:00-04:00
draft: false
originalDate: 2020-04-13T10:34:25-04:00
publishDate: 2020-04-15T07:00:00-04:00
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
description: "Learn how to write ArgumentParser tools with APIs that need Asynchronous Execution."
keywords:
 - swift
 - programming
 - apple
 - ArgumentParser
---

In the past four weeks we have explored many of the features available to us via ArgumentParser and how to use them. Here's a recap of everything we learned so far:

* We learned the [very basics](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part1/)
* We learned how to [validate user input](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part2/)
* We explored a way to [separate our tool into subcommands](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part3/).
* And finally, we learned how we can [improve our documentation pages](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part4/)

In this article, we will not explore a feature exposed to us via ArgumentParser. Instead, we will learn how to do something very essential: Creating tools that require asynchronous APIs.

# The Need for Asynchronous APIs.

If you have been programming for Apple platforms for a while, you have most likely used asynchronous APIs. `URLSession`, for example, is a fully asynchronous API, as network requests are unpredictable, they can take long, and therefore they need to be executed in a different thread.

The problem with this is that command line tools in general are very linear. They have a beginning point of execution, and an end. They don't really jump around different threads to do their job. In fact, if you ran a command line tool to do anything asynchronous, you'd see that it finishes up instantly without doing anything.

Below I have written an example that makes use of the [PokeApi](https://pokeapi.co). It is, for the most part, the same code I used when we talked about [Modern Background Tasks in iOS 13](https://www.andyibanez.com/posts/modern-background-tasks-ios13/).

```swift
struct Pokemon: Codable {
  struct Species: Codable {
    let name: String
  }
  
  struct Sprites: Codable {
    let backDefault: URL?
    let backShiny: URL?
    let frontDefault: URL?
    let frontShiny: URL?
    
    enum CodingKeys: String, CodingKey {
      case backDefault = "back_default"
      case backShiny = "back_shiny"
      case frontDefault = "front_default"
      case frontShiny = "front_shiny"
    }
  }
  
  let species: Species
  let sprites: Sprites
}

class PokeManager {
  static let urlSession = URLSession(configuration: .default)
  
  static func pokemon(id: Int,
                      completionHandler: @escaping (_ pokemon: Pokemon) -> Void) {
    let pokeUrl = buildPokemonURL(id: id)
    let task = urlSession.dataTask(with: pokeUrl) { (data, _, _) in
      let pokemon = try! JSONDecoder().decode(Pokemon.self, from: data!)
      DispatchQueue.main.async {
        completionHandler(pokemon)
      }
    }
    
    task.resume()
    
  }
  
  private static func buildPokemonURL(id: Int) -> URL {
    var urlComponents = URLComponents()
    urlComponents.scheme = "https"
    urlComponents.host = "pokeapi.co"
    urlComponents.path = "/api/v2/pokemon/\(id)"
    return urlComponents.url!
  }
}

struct Pokedex: ParsableCommand {
  
  static let configuration = CommandConfiguration(
    commandName: "pokedex",
    abstract: "Allows you to fetch info from a Pokémon with its Pokédex number.",
    discussion: "")
  
  @Argument(help: "number") var number: Int
  
  func run() throws {
    PokeManager.pokemon(id: number) { (pokemon) in
      self.printInfo(for: pokemon)
    }
  }
  
  func printInfo(for pokemon: Pokemon) {
    print("----------------------------------------------------------\n")
    print("INFO FOR POKÉMON: \(number)\n")
    print("ESPECIES: \(pokemon.species.name)\n")
    print("----------------------------------------------------------\n")
  }
}

Pokedex.main()
```

If you try to run this now, you will see the command line tool exits without printing anything at all:

```text
andyibanez@Andys-iMac Debug % ./MyCommandLinetool
Error: Missing expected argument '<number>'
Usage: pokedex <number>
andyibanez@Andys-iMac Debug % ./MyCommandLinetool 1
andyibanez@Andys-iMac Debug % 
```

## Making Asynchronous Tasks Behave Synchronously

As of right now, ArgumentParser has no tools to offer for us to be able to run asynchronous tasks such as network requests. So we have to figure out a way to do this ourselves. The concurrent APIs (the `Dispatch` APIs) provided to us by Cocoa and Cocoa Touch allow us to force execution of asynchronous tasks in the same process our command line tool is running.

But forcing the entire task to run asynchronously can be overkill. So what I like to do is to pause the execution of the thread that needs the resource, and continue it when another task finishes its execution. There are many ways to do this, but my favorite method is to use a Semaphore so we can force the static `pokemon(id:)` method to return the Pokémon with the `return` keyword instead of passing it in a completion handler.

Let's turn our attention to the method in question:

```
  static func pokemon(id: Int,
                      completionHandler: @escaping (_ pokemon: Pokemon) -> Void) {
    let pokeUrl = buildPokemonURL(id: id)
    let task = urlSession.dataTask(with: pokeUrl) { (data, _, _) in
      let pokemon = try! JSONDecoder().decode(Pokemon.self, from: data!)
      DispatchQueue.main.async {
        completionHandler(pokemon)
      }
    }
    
    task.resume()
    
  }
```

We want this to `return` the Pokémon traditionally. No completion handlers or anything like that. `URLSession` is fully asynchronous, so it doesn't have offer a way for us to do this either.

*Fun fact: You could still use the old `NSURLConnection` APIs and get synchronous behavior for network requests, but I prefer to use `URLSession` because it's more modern and we never know when `NSURLConnection` will go away - not to mention, it's good to know how to do this because not all asynchronous APIs are necessarily network-related.*

First change the signature to this:

```swift
static func pokemon(id: Int) -> Pokemon
```

We are about to do the magic that returns the Pokémon.

*Remember to deal with errors properly in a real world application. I'm skipping everything to do with error validation here. In a real app, you may want to return a tuple with an optional Pokémon and an Optional error, or handle errors in a different way.*

Now replace the entire body of the method with this:

```swift
let pokeUrl = buildPokemonURL(id: id)
var pokemon: Pokemon!

let semaphore = DispatchSemaphore(value: 0)

let task = urlSession.dataTask(with: pokeUrl) { (data, _, _) in
  pokemon = try! JSONDecoder().decode(Pokemon.self, from: data!)
  semaphore.signal()
}
task.resume()

semaphore.wait()

return pokemon
```

This implementation uses a Semaphore. This is not an article on concurrent programming, so let's just provide a very quick explanation of what a semaphore is, and how it works.

In concurrent programming, a semaphore controls access to a shared resource, ensuring that only one entity may access it at any given time.

When we call `wait` on the semaphore, it will wait until someone else calls `signal` on it. So essentially, this code will execute all the way down to `semaphore.wait`. The thread will pause until the completion handler is executed and assigns the Pokémon. After assigning the Pokémon, it will call `signal` so our previous thread can continue execution. because we will have a Pokémon before the `return` statement, we can just return the Pokémon.

Note that we did not switch threads or anything like that: All we did was to pause the thread that `pokémon(id:)` is executing in until the thread with the network request's completion handler is done executing.

You can read more about semaphores [here](https://medium.com/swiftly-swift/a-quick-look-at-semaphores-6b7b85233ddb).

Next change the implementation of your `run` method:

```swift
func run() throws {
	let pokemon = PokeManager.pokemon(id: number)
	printInfo(for: pokemon)
}
```

And we are done! Build and run your tool and it will properly fetch content from the network:

```text
andyibanez@Andys-iMac Debug % ./MyCommandLinetool 1
----------------------------------------------------------

INFO FOR POKÉMON: 1

ESPECIES: bulbasaur

----------------------------------------------------------
```

If your command line tool is fully asynchronous, you can just move the semaphore calls to the tool's `run` body.

# Conclusion

Command line tools that require asynchronous operations are very common. Even more so tools that connect to the network and do something with it. Thanks to Foundation's `Dispatch` APIs, we can `return` content that would otherwise need a completion handler. It's very useful to know this, because command line tools that need a network connection are very common.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.
