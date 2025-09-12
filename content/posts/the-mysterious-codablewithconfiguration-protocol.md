---
title: "The Mysterious CodableWithConfiguration Protocol"
date: 2021-09-29T07:00:00-04:00
originalDate: 2021-09-28T21:15:44-04:00
publishDate: 2021-09-29T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
categories:
 - development
tags:
 - swift
 - apple
 - programming
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - codable
 - json
 - no overview available
description: "Learn what the new CodableWithConfiguration type is and how you can use it to create better decoding and encoding code."
keywords:
 - swift
 - apple
 - programming
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - codable
 - json
 - no overview available
---

The Mysterious CodableWithConfiguration Protocol

Every year, at WWDC, Apple gives us a lot of new things to play around with. For the big new things, they prepare session videos and labs so developers can experiment with the new technologies and hopefully integrate them in their apps by the time the new OSes are out.

The "big things" are one thing, but Apple introduces *a bunch* of new APIs every year. Many (oh boy, **many**) of them do not get any coverage at WWDC at all. Not in a session, and sometimes not even a small mention of them anywhere.

Today I want to talk about a new API that did not get coverage but I really like: `CodableWithConfiguration`.

# Introducing CodableWithConfiguration

`CodableWithConfiguration` is not really a protocol on its own - it's actually a typealias for the union of the `EncodableWithConfiguration` and `DecodableWithConfiguration` protocols. If you have worked with `Codable` before, this all my sound familiar - That's because `Codable` is really a typealias for `Encodable & Decodable`. If you already know how to use `Codable`, you can get started with this new shiny API very quickly. `CodableWithConfiguration` allows you to do exact same thing as plain old `Codable` - Serialize objects to and from different formats, such as JSON.

But what, exactly, is `CodableWithConfiguration`?

`CodableWithConfiguration` allows us to inject a configuration object into the decoding and encoding processes of our codable types. These configuration allow you to have a bit more control over these processes without losing much flexibility. The configurations can do anything: Mutate objects, selectively decode or encode some keys, transform them... The sky is the limit. Configuration objects can be, after all, *anything* you want.

To make this concept clearer, I will show you some examples of some use cases I have thought of, so hopefully you can start using `CodableWithConfiguration` right away.

Before continuing, keep in that this protocol was added on WWDC2021 to all Apple's platforms, so you will need to target iOS 15, macOS 12, and so on.

## Using CodableWithConfiguration

To work with these examples, assume we have the following JSON objects:

`User.json`:

```
{
  "userId": 1,
  "username": "AndyIbanez",
  "avatarURL": "https://pbs.twimg.com/profile_images/1403463750406098947/-gU-Ofaa_400x400.jpg",
  "biography": "iOS Developer writing nifty apps.",
  "interests": [
    "Programming",
    "Pullip",
    "The Legend of Zelda"
  ],
  "videogames": []
}
```

`Videogame.json`:

```
{
  "videogameId": 1,
  "title": "The Legend of Zelda: Ocarina of Time",
  "developer": "Nintendo",
  "publisher": "Nintendo",
  "personalRating": 10,
  "hoursPlayed": 128
}
```

The idea is that we have a service that allows users to create their profiles and keep track of their videogames. Users can Update their avatar, biography, interests, and videogames. Web services don't commonly allow you to edit your username, much less your user ID.

The info users can update from their videogames is their personal rating and the total hours they played them. The other info (videogame title, videogame ID, developer and publisher) belongs to the service, in a database somewhere, so it's not really user editable.

To convert them into objects, your first instinct may be to create the following types:

```swift
struct User: Codable {
  let userId: Int?
  let username: String?
  var avatarURL: URL?
  var biography: String?
  var interests: [String]?
  var videogames: [Videogame]?
}
```

```swift
struct Videogame: Codable {
  let videogameId: Int?
  let title: String?
  let developer: String?
  let publisher: String?
  var personalRating: Int?
  var hoursPlayed: Int?
}
```

And this works fine, but configurations allow us to do some interesting things.

### Transforming properties based on configurations

Configurations are really flexible. Just the fact you can define *any* object as your configuration object opens a world of possibilities. So from here on out, I will show you why these configuration objects are useful using examples, and we will start with simple property transformation.

Suppose the API you are working with allows you to search for users. In a search view, you do not care about showing all the user info - you may care about having the username and avatar always visible. You can show a portion of the biography to help your users locate the right user they are searching for, but showing entire biographies could make your UI look funny. You could also show the first two interests in the search view, but definitely not all of them. You may also keep the user ID (although not visible) to use it as a reference for other calls.

If you wanted to achieve this with raw `Codable`, you would have to transform the properties *after* you have the objects. If the biography is too long or there are too many interests, it's hard to know at decoding time because even if you implemented `init(from:)` yourself, you would have to hardcode all the logic there, and instead of having a reusable `User` object across your app, you would end up having a `UserSearch` object for the search results, a `UserProfile` object to display their entire profile, and so on. Something like this:

```swift
// UserSearch.swift
// Notice that we don't have the videogames property, because it's not used in search results.
struct UserSearch: Codable {
  let userId: Int?
  let username: String?
  let avatarURL: URL?
  let biography: String?
  let interests: [String]?
  
  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    userId = try container.decodeIfPresent(Int.self, forKey: .userId)
    username = try container.decodeIfPresent(String.self, forKey: .username)
    avatarURL = try container.decodeIfPresent(URL.self, forKey: .avatarURL)
    
    if let tempBiography = try container.decodeIfPresent(String.self, forKey: .biography) {
      // The biography will be truncated to a max of 20 characters.
      biography = String(tempBiography.prefix(20))
    } else {
      biography = nil
    }
    
    if let tempInterests = try container.decodeIfPresent([String].self, forKey: .interests) {
      // We will keep a maximum of two interests.
      interests = Array(tempInterests.prefix(2))
    } else {
      interests = nil
    }
  }
}
```

In the code above, we are hard coding the logic to limit biographies and interests in the decoder initializer. This forces us to write more code, but that is not necessarily a bad thing. The main problem is the biography length and interests have their limitations hard coded and cannot be changed.

```swift
struct UserProfile: Codable {
  let userId: Int?
  let username: String?
  let avatarURL: URL?
  let biography: String?
  let interests: [String]?
  let videogames: [Videogame]
}
```

The code above is much easier, but it's still a copy/paste of all the properties of the `UserSearch` object. It has one additional property for videogames, but since this displays complete user profiles, we do not need to truncate the biography or limit the number of interests.

Clearly, in this situation, prior to to `CodableWithConfiguration`, it's much better to just apply the transformations after decoding, maybe even at the moment you show your UI.

```swift
// We decoded a User object called `user`
userBio.text = user.biography.prefix(20)
```

The way we do this with `CodableWithConfiguration` is prettier. We will start by creating a `UserConfiguration` object that will hold properties to dynamically limit biography length and a maximum number of interests.

```swift
struct UserConfiguration {
  // If nil, there is no limitation to biographies length.
  let biographyMaxLength: Int?
  
  // If nil, there is no limitations for interests
  let maxInterests: Int?
}
```

Next, we will grab the `User` object we created in the section above, and we will make it conform to both `Codable` and `CodableWithConfiguration`

```swift
struct User: Codable, CodableWithConfiguration {
	//...
}
```

Do note that by making the class conform with Codable, you will get the `CodingKeys` for free. If you only conform to `CodingWithConfiguration` you will need to provide the enum yourself.

Next, we need to implement two methods:

```swift
init(from decoder: Decoder, configuration: UserConfiguration) throws {

}

func encode(to encoder: Encoder, configuration: UserConfiguration) throws {

}
```

`CodableWithConfigurations` offers the `DecodingConfiguration` and `DecodingConfiguration` associated types, and since we are providing the initializers ourselves, the compiler will infer that the typealias for both is `UserConfiguration`. For now, we will only use the decoder, we will add a `fatalError()` call to `encode(to:configuration)` since we don't plan using it (an example of how you can use it will be shown below).

We will now implement `init(from:configuration)` so that it uses our configuration object to dynamically limit the biography, the interests, or both:

```swift
init(from decoder: Decoder, configuration: UserConfiguration) throws {
  let container = try decoder.container(keyedBy: CodingKeys.self)
  userId = try container.decodeIfPresent(Int.self, forKey: .userId)
  username = try container.decodeIfPresent(String.self, forKey: .username)
  avatarURL = try container.decodeIfPresent(URL.self, forKey: .avatarURL)
  
  biography = try {
    let bio = try container.decodeIfPresent(String.self, forKey: .biography)
    if let maxLength = configuration.biographyMaxLength, let bio = bio {
      // We will use our configuration object to dynamically truncate the biography to a length. Neat!
      return String(bio.prefix(maxLength))
    }
    return bio
  }()
  
  interests = try {
    let interests = try container.decodeIfPresent([String].self, forKey: .interests)
    if let maxInterests = configuration.maxInterests, let interests = interests {
      // Using the configuration object to limit the max number of interests.
      return Array(interests.prefix(maxInterests))
    }
    return interests
  }()
  
  videogames = try container.decodeIfPresent([Videogame].self, forKey: .videogames)
}
```

It looks like a mouthful, but I want you to sit down and appreciate the power of `CodingWithConfiguration`. We can now configure our object at decoding time, and we no longer need to do these transformations when showing the UI or creating different `User*` codable objects to achieve the same goal. The same user object we use to limit the amount of data displayed in the Search view, is the same object we can use to display a full user profile, in two different views.

### Choosing the properties that should be included in a JSON

Another use case for `CodableWithConfiguration` is to limit the keys that should be decoded or encoded. This can be useful when you have an API that returns you a JSON, and then expects that JSON back with some changes for some operation.

Suppose you want to allow users to edit their videogame data (`personalRating` and `hoursPlayed`) and it expects a JSON with the same structure as `Videogame`. When we are updating a videogame, we only want to give the API the `personalRating`, `hoursPlayed`, and the `videogameId`. The ID is necessary, otherwise the webservice won't know what videogame we want to update. On the other hand, we don't really need to send the `title`, `developer`, and `publisher` because the service owns this data and it's not user editable. So to save the user a bit of their data plan, we can filter out those properties only when we are performing an update.

Start by adding the `CodableWithConfiguration` conformance to videogame.

```swift
struct Videogame: Codable, CodableWithConfiguration {}
```

You will also want to explicit declare the coding keys, otherwise they stay private and there unaccessible:

```swift
enum CodingKeys: String, CodingKey, CaseIterable {
  case videogameId
  case title
  case developer
  case publisher
  case personalRating
  case hoursPlayed
}
```

This time, we will implement `encode(to:configuration)` method.

```swift
func encode(to encoder: Encoder, configuration: VideogameConfiguration) throws {
  var container = encoder.container(keyedBy: CodingKeys.self)
  
  if configuration.codingKeys.contains(.videogameId) {
    try container.encode(videogameId, forKey: .videogameId)
  }
  
  if configuration.codingKeys.contains(.title) {
    try container.encode(title, forKey: .title)
  }
  
  if configuration.codingKeys.contains(.developer) {
    try container.encode(developer, forKey: .developer)
  }
  
  if configuration.codingKeys.contains(.publisher) {
    try container.encode(publisher, forKey: .publisher)
  }
  
  if configuration.codingKeys.contains(.personalRating) {
    try container.encode(personalRating, forKey: .personalRating)
  }
  
  if configuration.codingKeys.contains(.hoursPlayed) {
    try container.encode(hoursPlayed, forKey: .hoursPlayed)
  }
}
```

This time, we are simply checking to see if the configuration contains a given key, and if it does, we include it in the resulting JSON.

We will also add two helper static properties on `VideogameConfiguration` to have an easy reference to the object's coding keys:

```swift
struct VideogameConfiguration {
  let codingKeys: Set<Videogame.CodingKeys>
  static let allKeys = Videogame.CodingKeys.allCases
  static let userWriteable: Set<Videogame.CodingKeys> = Set(arrayLiteral: .hoursPlayed, .personalRating,  .videogameId)
}
```

For `Videogame`s (and you could do this for `User` as well), you can implement `decode(from:configuration)` so that it checks each key to see if you need it:

```swift
init(from decoder: Decoder, configuration: VideogameConfiguration) throws {
  let container = try decoder.container(keyedBy: CodingKeys.self)
  
  if configuration.codingKeys.contains(.videogameId) {
    videogameId = try container.decodeIfPresent(Int.self, forKey: .videogameId)
  } else {
    videogameId = nil
  }
  
  if configuration.codingKeys.contains(.title) {
    title = try container.decodeIfPresent(String.self, forKey: .title)
  } else {
    title = nil
  }
  
  if configuration.codingKeys.contains(.developer) {
    developer = try container.decodeIfPresent(String.self, forKey: .developer)
  } else {
    developer = nil
  }
  
  if configuration.codingKeys.contains(.publisher) {
    publisher = try container.decodeIfPresent(String.self, forKey: .publisher)
  } else {
    publisher = nil
  }
  
  if configuration.codingKeys.contains(.personalRating) {
    personalRating = try container.decodeIfPresent(Int.self, forKey: .personalRating)
  } else {
    personalRating = nil
  }
  
  if configuration.codingKeys.contains(.hoursPlayed) {
    hoursPlayed = try container.decodeIfPresent(Int.self, forKey: .hoursPlayed)
  } else {
    hoursPlayed = nil
  }
}
```

I also want to take this chance to take you to the `User`s `decode(from:configuration)` method before we move on. As the last line of the method, you have this call:

```swift
videogames = try container.decodeIfPresent([Videogame].self, forKey: .videogames)
```

And this works fine, but if you need to, you can actually pass a configuration to decode videogames differently as well. To do this, we can add an optional `videogameConfiguration` property to `UserConfiguration`:

```swift
struct UserConfiguration {
  // If nil, there is no limitation to biographies length.
  let biographyMaxLength: Int?
  
  // If nil, there is no limitations for interests
  let maxInterests: Int?
  
  /// The configuration we will use for Videogames
  let videogameConfiguration: VideogameConfiguration?
}
```

And then we can simply check for it at decoding time to see if we want to decode by default with `Codable`'s encode method, or with `CodableWithConfiguration`'s method:

```swift
if let vgConfig = configuration.videogameConfiguration {
  videogames = try container.decodeIfPresent([Videogame].self, forKey: .videogames, configuration: vgConfig)
} else {
  videogames = try container.decodeIfPresent([Videogame].self, forKey: .videogames)
}
```

You can do the same with `Encoder`.

## Applying our Configurations to Codable Objects

So far, I have shown you how you can create your configurations, but how do you actually use them?

First, I have some bad news for you. As of now, these are only useful when you are dealing with "envelope" JSON objects. That is to say, if you have a JSON that looks like this:

```
{
	"resultCount": 30,
	"totalPages": 5,
	"resultsPerPage": 5,
	"currentPage": 3,
	"videogames: []
}
```

You have an envelope object. An envelope object is a top-level JSON that wraps one or more complex JSON objects (in this case, `videogames` is our "wrapped" object, which contains an array of `Videogame`s).

You cannot apply configurations to top level objects directly.

```
{
  "videogameId": 1,
  "title": "The Legend of Zelda: Ocarina of Time",
  "developer": "Nintendo",
  "publisher": "Nintendo",
  "personalRating": 10,
  "hoursPlayed": 128
}
```

In this example, the entire object is our videogame. There's no wrapped object to speak of, so if you wanted to apply a configuration to a top-level `Videogame`, you sadly can't. This is because neither `JSONEncoder` or `JSONDecoder` have a variation of their decoding/encoding methods that take a configuration. If you are interested in this functionality, please dupe my feedback: FB9662199.

### The CodableConfiguration Property Wrapper

We have finally arrived to the section were we talk about how we can actually apply our configurations.

There is a property wrapper called [`@CodableConfiguration`](https://developer.apple.com/documentation/foundation/codableconfiguration). You use this property wrapper with codable objects.

In order to use this property wrapper, you need to create configuration providers and provide configurations for encoding and decoding. These provider objects must conform to `DecodingConfigurationProviding` and `EncodingConfigurationProviding` (for some reason, there is no typealias for `DecodingConfigurationProviding & EncodingConfigurationProviding`). To fulfill their requirements you need to provide static properties with the configuration you want to use. Going back to our user search example, we can create the following provider used exclusively for `User` objects found within other objects.

```swift
struct UserConfigurationSearchProviding: DecodingConfigurationProviding, EncodingConfigurationProviding {
  static var decodingConfiguration: UserConfiguration = .init(biographyMaxLength: 20, maxInterests: 2, videogameConfiguration: nil)
  static var encodingConfiguration: UserConfiguration = .init(biographyMaxLength: 20, maxInterests: 2, videogameConfiguration: nil)
}
```

Unfortunately, you need to provide both configurations, even if you don't use them - recall attempting to encode a User will result in `fatalError()` being called. For now, we will simply use the same configuration we use for decoding.

The result object looks like this:

```swift
{
  "resultCount": 1,
  "totalPages": 1,
  "currentPage": 1,
  "resultsPerPage": 5,
  "users": [
    {
      "userId": 1,
      "username": "AndyIbanez",
      "avatarURL": "https://pbs.twimg.com/profile_images/1403463750406098947/-gU-Ofaa_400x400.jpg",
      "biography": "iOS Developer writing nifty apps. I was born and raised in La Paz, Bolivia many moons ago.",
      "interests": [
        "Programming",
        "Pullip",
        "The Legend of Zelda"
      ],
      "videogames": [
        {
          "videogameId": 1,
          "title": "The Legend of Zelda: Ocarina of Time",
          "publisher": "Nintendo",
          "developer": "Nintendo",
          "personalRating": 10,
          "hoursPlayed": 900
        },
        {
          "videogameId": 2,
          "title": "The Legend of Zelda: Majora's Mask",
          "publisher": "Nintendo",
          "developer": "Nintendo",
          "personalRating": 10,
          "hoursPlayed": 800
        }
      ]
    }
  ]
}
```

And so, in order to decode it with the configuration object, you can do something like this:

```swift
struct UserSearch: Codable {
  let resultCount: Int
  let totalPages: Int
  let resultsPerPage: Int
  let currentPage: Int
  @CodableConfiguration(from: UserConfigurationSearchProviding.self) var users = [User]()
}
```

Beautiful! Injecting the provider this way will allow us to reuse the same `User` object and mutate it as necessary without having to delegate the work to upper layers of our app.

```swift
let users = try! JSONDecoder().decode(UserSearch.self, from: jsonData!)
print(users.users.first!.biography) // "iOS Developer writin"
print(users.users.first!.interests) // ["Programming", "Pullip"]
```

Now suppose our user wants to update some data in their videogames. You can start creating the configuration provider like so:

```swift
struct VideogameConfigurationUpdateProviding: DecodingConfigurationProviding, EncodingConfigurationProviding {
  static var decodingConfiguration: VideogameConfiguration = VideogameConfiguration(codingKeys: VideogameConfiguration.allKeys)
  static var encodingConfiguration: VideogameConfiguration = VideogameConfiguration(codingKeys: VideogameConfiguration.userWriteable)
}
```

The model object has a single property called `updatedVideogames`.

```swift
struct VideogameUpdateRequest: Codable {
  @CodableConfiguration(from: VideogameConfigurationUpdateProviding.self) var updatedVideogames = [Videogame]()
  
  init(videogames: [Videogame]) {
    self.updatedVideogames = videogames
  }
}
```

When this gets converted into JSON, it will discard all the properties of the videogames except for `videogameId`, `hoursPlayed`, and `personalRating`, which is the bare minimum we need to update their data:

```swift
let jsonString =
"""
{
  "resultCount": 1,
  "totalPages": 1,
  "currentPage": 1,
  "resultsPerPage": 5,
  "users": [
    {
      "userId": 1,
      "username": "AndyIbanez",
      "avatarURL": "https://pbs.twimg.com/profile_images/1403463750406098947/-gU-Ofaa_400x400.jpg",
      "biography": "iOS Developer writing nifty apps. I was born and raised in La Paz, Bolivia many moons ago.",
      "interests": [
        "Programming",
        "Pullip",
        "The Legend of Zelda"
      ],
      "videogames": [
        {
          "videogameId": 1,
          "title": "The Legend of Zelda: Ocarina of Time",
          "publisher": "Nintendo",
          "developer": "Nintendo",
          "personalRating": 10,
          "hoursPlayed": 900
        },
        {
          "videogameId": 2,
          "title": "The Legend of Zelda: Majora's Mask",
          "publisher": "Nintendo",
          "developer": "Nintendo",
          "personalRating": 10,
          "hoursPlayed": 800
        }
      ]
    }
  ]
}
"""

let jsonData = jsonString.data(using: .utf8)
let users = try! JSONDecoder().decode(UserSearch.self, from: jsonData!)

// My games
var games = users.users.first!.videogames!
var firstGame = games[0]
firstGame.personalRating = 11
firstGame.hoursPlayed = 1000

var secondGame = games[1]
secondGame.personalRating = 11
secondGame.hoursPlayed = 900

let allGames = [firstGame, secondGame]
let gamesToUpdate = VideogameUpdateRequest(videogames: allGames)
print(gamesToUpdate)
```

And as you can see, the JSON contains very few properties, as it is expected:

```
{
  "updatedVideogames": [
    {
      "personalRating": 11,
      "hoursPlayed": 1000,
      "videogameId": 1
    },
    {
      "personalRating": 11,
      "hoursPlayed": 900,
      "videogameId": 2
    }
  ]
}
```

# Conclusion

`CodableWithConfiguration` is a very interesting object, and one of my favorite new additions to WWDC2021 for sure. Being able to inject configurations to have control over the encoding and decoding processes opens up a world of possibilities and cleaner code. This new API also makes use of property wrappers, which are a personal favorite feature of mine added to Swift in the past few years. While it's hard to use them with top-level objects, the power is still there, and hopefully in the future, we will see versions of `JSONEncoder` and `JSONDecoder` that can take configuration providers to work with root objects.