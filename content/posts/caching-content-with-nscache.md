---
title: "Caching Content With NSCache"
date: 2019-08-28T12:08:07-04:00
draft: true
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - macos
 - tvos
 - watchos
 - ipados
categories:
 - development
description: "Learn to cache content on Apple's Platforms with NSCache, a high-level native caching API."
keywords:
 - swift
 - caching
 - ios
 - tvos
 - ipados
 - watchos
---

When we are working with apps on iOS, iPadOS, macOS, watchOS, or TVOS, it's possible that at some point we will have to store and retrieve a lot of temporary data throughout the lifecycle of our software. Depending on our needs, we may need to cache data on disk and manually manage it ourselves, or we may only need it to cache it in memory. In the case of the latter, Apple offers `NSCache`, a mutable collection that lets us cache files in memory using key-value pairs.

`NSCache` is really nice for a few reasons:

* It stores data in memory only. If our app gets killed, this memory is freed up and it's not persisted to disk.
* The key-value pair mechanism lets us very easily set and get cached content. Very similar to what we would do with a `Dictionary`. Unlike a dictionary, the keys are not copied, so it's slightly more memory efficient.
* We can set automatic eviction conditions to help `NSCache` delete objects automatically. We can also manually evict objects if we need to.
* It is multi-threading friendly. We can read and write the cache without having to manage our cache object ourselves.

And there is just one reason it is not perfect:

* It's an Objective-C API, so you will end up doing some casting, even when working with basic objects such as strings.

Use `NSCache` to store temporary objects that are expensive to create, but can be re-created if necessary. Suppose we have an app that downloads a lot of images to display to the user dynamically and they are very big - downloading them takes a long time and they consume a lot of data. It would be bad to re-download them these images every time the user needed them, so we can cache them instead. If at some point the system starts demanding memory, the system can just remove these images and we can re-download them if necessary.

`NSCache` is available in all of Apple's Platforms: watchOS, iOS, iPadOS, macOS, and TVOS.

# NSCache Basics

## Creating a NSCache Object

The constructor of the `NSCache` object takes two generic objects: The key type, and the cached object type. We can optionally give it a name to identify it later.

```swift
let cache = NSCache<NSString, UIImage>()
cache.name = "Remote Image Cache"
```

This API has its roots in the Objective-C days, and as such the generic parameters are constrained to conform to `AnyObject`, meaning that we cannot use `struct`s and must uses `class`es instead. For that reason we must use `NSString` instead of `String`. Both our keys and objects can be of any type as long as they are classes. In this example we chose strings for the keys, and images for the objects.

## Storing Objects

Storing an object is as easy as calling the cache's `setObject` method.

```swift
let webImage = UIImage(named: "pullip_doll.png")!
cache.setObject(webImage, forKey: "top banner")
```

There is also an overloaded `setObject(object:forKey:cost:)` method, which we will talk about in a bit.

*(I'd love it if the API offered a subscript for this kind of task, but sadly that's not the case.)*

## Retrieving Objects

Retrieving objects is just as easy. There's just one method called `object(forKey:)`. This method returns an optional `ObjectType` (in our case, an optional `UIImage`), so we can easily check if the object exists. Whether the object no longer exists or it has been evicted, it will return `nil`.

```swift
if let webImage = cache.object(forKey: "top banner") {
		// Do something with webImage
    print("The object is still cached")
} else {
    print("Web image went away")
}
```

*(Just like before there's no native subscript for this.)*

## Removing Objects

Deleting objects does not possess any kind of complexity, and there's methods to evict either a single object or the entire cache.

To remove a single object, just call the cache's `removeObject(forKey:)` method.

```swift
cache.removeObject(forKey: "top banner")
```

And to remove all the objects, simply call `removeAllObjects()` on the cache.

```swift
cache.removeAllObjects()
```

# Automatic Eviction Conditions

Having manual control over the cache is important and it's going to be enough for many cases, but `NSCache` allows us to set conditions to automatically clean after itself. We can constrain it to hold a limited amount of objects, and we can specify a maximum "cost".

Even when we don't set any eviction conditions, `NSCache` will start deleting objects if the system is really hungry for memory, so we cannot count on our objects always being there, even we don't set any eviction conditions ourselves.

## Limiting the Amount of Objects in the Cache

To limit the amount of objects our cache should hold, set the `countLimit` property to anything higher than 0. `0` means no limit, so the cache will keep storing objects indefinitely (unless the system really needs some memory, that is).

```swift
cache.countLimit = 10
```

What a good size is depends strictly on our application. If we are dealing with big images, we can set a low number here, but in the case of something smaller, such as strings, it can probably be way higher.

It's worth noting that this is not a strict limit. The eviction of objects is governed by the implementation of the cache. If the cache goes over the limit, it may remove objects instantly, at a later moment, or possibly even never. It will all depend on the needs of the system at a given time.

## Setting a Maximum Cost

### Definition of Cache Object Cost

The "cost" of an object in the cache is a bit abstract, and it depends on the context in which a cache is operating.

Let's go back to the example of storing images in the cache. We can define the "cost" of an image as its size in bytes. A bigger image will have a bigger cost. We could find a different definition, such as its size in dimensions (it's weight and height).

If you wanted to store strings, you could define the "cost" based on the number of characters in each string. So the string `"Pullip Classical Alice"` (22 characters) has a bigger cost than `"Pullip Alura"` (12 characters).

### Limiting the Maximum Total Cost of the Cache

To set the maximum cost, set the `totalCostLimit` property of the cache. This number is an `Int`, and once again, what exactly it represents depends on the context of each cache.

```swift
// For our image cache, we will set a maximum cost of 50,000,000 bytes, or 50 megabytes.
cache.totalCostLimit = 50_000_000
```

Now, when we want to add objects along with their cost, we can use the `setObject(object:forKey:cost:)` method we mentioned above.

```swift
// Convert the image to Data.
if let topBannerData = webImage.pngData() {
    // The cost of our image is its size in bytes.
    cache.setObject(webImage, forKey: "top banner", cost: topBannerData.count)
}
```

Just like setting the maximum total objects, though, this is not a strict limit, and the cache will decide what to do with the objects once the limit is surpassed. If it needs to start evicting objects, it will start deleting some until the total cost of the cache is under the `totalCostLimit`. Keep in mind that the order in which the objects will be removed is random. We cannot, for example, expect the cache to start removing the biggest cost objects first (in our example, the biggest images), and there's no way to enforce a specific order.

# The NSDiscardableContent Protocol

The `NSDiscardableContent` protocol can be implemented when an object has subcomponents that can be discarded when not being used.

Suppose we have a class `Person` that looks like this:

```swift
class Person {
    let firstName: String
    let lastName: String
    var avatar: UIImage? = nil
    
    init(firstName: String, lastName: String, avatar: UIImage?) {
        self.firstName = firstName
        self.lastName = lastName
        self.avatar = avatar
    }
}
```

We want to cache this, but the `firstName` and `lastName` properties are probably too small to care about them persisting for a long time. On the other hand, the `avatar` can be big, so we want to remove only the `avatar` property when the system needs it. In this case, `Person` is a content-object, and the `avatar` property is the subcomponent that can be discarded.

`NSCache` allows us to do this by implementing the `NSDiscardableContent` in our objects.

`NSDiscardableContent` works with a simple variable counter system. When the memory is being read or is currently needed, its counter will have a value of `1`. If it's not needed at all and is not being used, the counter will be `0`. When a new ``NSDiscardableContent` is created, it's counter value starts with `1`. We will see how we can make use of this to help `NSCache` manage our `Person` class.

When we conform to `NSDiscardableContent`, there's four methods we must adopt:

```swift
// True if the content is still available and have been successfully accessed.
func beginContentAccess() -> Bool {
}

// Called when the content is no longer being accessed.
func endContentAccess() {
}

// If our counter is 0, we can discard the image.
func discardContentIfPossible() {
}

// True if the content has been discarded.
func isContentDiscarded() -> Bool {
}
```

We can implement `Person` conforming to the protocol the following way:

```swift
class Person: NSDiscardableContent {
    let firstName: String
    let lastName: String
    var avatar: UIImage? = nil
    
    // Our counter variable
    var accessCounter = true
    
    init(firstName: String, lastName: String, avatar: UIImage?) {
        self.firstName = firstName
        self.lastName = lastName
        self.avatar = avatar
    }
    
    // MARK: - NSDiscardableContent
    
    func beginContentAccess() -> Bool {
        if avatar != nil {
            accessCounter = true
        } else {
            accessCounter = false
        }
        return accessCounter
    }
    
    func endContentAccess() {
        accessCounter = false
    }
    
    func discardContentIfPossible() {
        avatar = nil
    }
    
    func isContentDiscarded() -> Bool {
        return avatar == nil
    }
}
```

Now we can create a cache of `Person`s. But there is one more thing we need to do.

By default, `NSCache` will evict all the objects it contains. In our case, it will discard `Person`s as necessary, and not just their avatar. To change this, set the cache's `evictsObjectsWithDiscardedContent` property to `false`.

```swift
cache.evictsObjectsWithDiscardedContent = false
```

This property, whose default value is `true`, controls whether entire objects from the cache will be removed or just their discardable content. Setting it to `false` will ensure it just discards the `avatar`s and not whole `Person`s.

We can new create a new cache object of `Person`s and add objects to it.

```swift
let cache = NSCache<NSString, Person>()
cache.name = "Person Cache"
cache.evictsObjectsWithDiscardedContent = false

let andy = Person(firstName: "Andy", lastName: "Ibanez", avatar: UIImage(named: "silight.png"))
cache.setObject(andy, forKey: "me")
```

Now, when the cache starts deleting `Person`s, it will only delete their avatars.

# The NSCacheDelegate Protocol

To finish off this post, we can talk about the `NSCacheDelegate` protocol, which allows to see what a specific cache is doing. Currently, the delegate only has one method, `cache(_:willEvictObject)`, which allows us to know when an object is being removed.

```swift
func cache(_ cache: NSCache<AnyObject, AnyObject>, willEvictObject obj: Any) {
    if let person = obj as? Person {
        print("Cache \(cache.name) will evict person \(person.firstName) \(person.lastName)")
    }
}
```

When an object is about to be deleted, we will be notified, which allows us to take some action. For now, we will just print who the person is that is being evicted.

*(As is the case with the other API examples above, this comes from Objective-C, so we have to do some casting.)*

# Conclusion

`NSCache` is a good API to cache content that you only need in memory. You can both control the contents manually, or you can set conditions to allow the cache to manage itself. Being an Objective-C object at its core, it has some quirks to work with, but it's still very easy to use.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.