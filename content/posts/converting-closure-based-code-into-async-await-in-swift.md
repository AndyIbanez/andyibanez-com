---
title: "Converting closure-based code into async/await in Swift"
date: 2021-06-23T07:00:00-04:00
originalDate: 2021-06-17T22:47:49-04:00
publishDate: 2021-06-23T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
categories:
 - article series
 - modern concurrency article series
 - modern concurrency in swift article series
 - development
tags:
 - swift
 - apple
 - programming
 - ios
 - concurrency
 - async
 - await
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - multithreading
description: "Learn about checked continuations and converting closure and delegate-based code into async/await in Swift."
keywords:
 - swift
 - apple
 - programming
 - ios
 - concurrency
 - async
 - await
 - ios
 - macos
 - ipados
 - watchos
 - wwdc2021
 - multithreading
---

*This article is part of my [Modern Concurrency in Swift](/posts/modern-concurrency-in-swift-introduction/) article series.*

*This article was originally written creating examples using Xcode 13 beta 1. The article, code samples, and provided sample project have been updated for Xcode 13 beta 3.*

###### Table of Contents

1. [Modern Concurrency in Swift: Introduction](/posts/modern-concurrency-in-swift-introduction/)
2. [Understanding async/await in Swift](/posts/understanding-async-await-in-swift/)
3. **Converting closure-based code into async/await in Swift**
4. [Structured Concurrency in Swift: Using async let](/posts/structured-concurrency-in-swift-using-async-let/)
5. [Structured Concurrency With Group Tasks in Swift](/posts/structured-concurrency-with-group-tasks-in-swift/)
6. [Introduction to Unstructured Concurrency in Swift](/posts/introduction-to-unstructured-concurrency-in-swift/)
7. [Unstructured Concurrency With Detached Tasks in Swift](/posts/unstructured-concurrency-with-detached-tasks-in-swift/)

<hr>

*To better benefit from this article, you should be familiar with async/await. If you aren't, feel free to read the first part of this article series: [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/).*

*I was debating whether this article should be its own or if its contents should be appended to Introducing async/await in Swift. I decided to make the previous article shorter in an attempt to not overload the articles with information, and to hopefully make it easier to understand these API with smaller articles.*

Last week, we had a long discussion on async/await. We contrasted how it compares to callbacks, and we showed examples that hopefully convinced async/await is really neat.

We are just one step away from actual concurrency. Before we dive in into concurrency - with *structured concurrency* - next week, I want to show you how you can convert closure-based and delegate-based code into async/await code. The idea behind this article is to give you all the tools so that you can start adopting async/await in your projects, baby steps at a time.

If you are a library vendor, you will be able to provide async/await code for all your closure-based APIs, so not only will you be able to start using it for your uses, you will be able to ship async/await to your users.

If you are not a library vendor, but you do have an app in production, it's likely that your own app is using asynchronous code that notifies you via callbacks. If you want to start migrating that project, you can start by implementing `async` versions of your async methods. If you are using a third party library that is not offering async/await versions of their calls, you can easily provide your own.

# Understanding Continuations

If you have read the [first part](https://www.andyibanez.com/posts/understanding-async-await-in-swift/) of this article series, you may remember what a Continuation is, but let's have a quick refresher before we move on.

A continuation is simply what happens after an async call. When you are using async/await, the continuation is easy to understand: Everything below an `await` call, is a continuation.

Consider the following example:

```swift
func downloadMetadata(for id: Int) async throws -> ImageMetadata {
    let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(id).json")!
    let metadataRequest = URLRequest(url: metadataUrl)
    let (data, metadataResponse) = try await URLSession.shared.data(for: metadataRequest)
    guard (metadataResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.invalidMetadata
    }

    return try JSONDecoder().decode(ImageMetadata.self, from: data)
}
```

In this example, the keyword `await` will (may) trigger a data download task in a different thread. Everything underneath `await` (that is, starting on the line with a `guard`), is a *continuation*.

Continuations are not limited to the async/await APIs. When you are using closure-based async APIs, a continuation is everything called within your completion handlers.

```swift
let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).json")!
let metadataTask = URLSession.shared.dataTask(with: metadataUrl) { data, response, error in
    guard let data = data, let metadata = try? JSONDecoder().decode(ImageMetadata.self, from: data),  (response as? HTTPURLResponse)?.statusCode == 200 else {
        completionHandler(nil, ImageDownloadError.invalidMetadata)
        return
    }
    let detailedImage = DetailedImage(image: image, metadata: metadata)
    completionHandler(detailedImage, nil)
}
metadataTask.resume()

```

This is a closure version of the code above. Once again, the continuation starts at the `guard`. The main difference is the completion handler version has a flow that is harder to follow.

## Introducing explicit continuations

Swift provides a few methods we can use to convert callback-based code into async/await: `withCheckedContinuation` and `withCheckedThrowingContinuation`. The difference between the two is the latter is used for code that throws errors. I call these methods *explicit continuations*.

Suppose you have a completion handler version of the `downloadMetadata(for:)` method declared above:

```swift
// MARK: - Definitions

struct ImageMetadata: Codable {
    let name: String
    let firstAppearance: String
    let year: Int
}

struct DetailedImage {
    let image: UIImage
    let metadata: ImageMetadata
}

enum ImageDownloadError: Error {
    case badImage
    case invalidMetadata
}

// MARK: - Functions

func downloadImageAndMetadata(
    imageNumber: Int,
    completionHandler: @escaping (_ image: DetailedImage?, _ error: Error?) -> Void
) {
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).png")!
    let imageTask = URLSession.shared.dataTask(with: imageUrl) { data, response, error in
        guard let data = data, let image = UIImage(data: data), (response as? HTTPURLResponse)?.statusCode == 200 else {
            completionHandler(nil, ImageDownloadError.badImage)
            return
        }
        let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/\(imageNumber).json")!
        let metadataTask = URLSession.shared.dataTask(with: metadataUrl) { data, response, error in
            guard let data = data, let metadata = try? JSONDecoder().decode(ImageMetadata.self, from: data),  (response as? HTTPURLResponse)?.statusCode == 200 else {
                completionHandler(nil, ImageDownloadError.invalidMetadata)
                return
            }
            let detailedImage = DetailedImage(image: image, metadata: metadata)
            completionHandler(detailedImage, nil)
        }
        metadataTask.resume()
    }
    imageTask.resume()
}
```

And suppose you are not the original author of it, and it's closed source, preventing you from modifying it directly. If you wanted to start your async/await migration with this method, the simplest way to do it would be by wrapping a call to `downloadImageAndMetadata(for:imageNumber:completionHandler)` inside the `withCheckedThrowingContinuation` method.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    return try await withCheckedThrowingContinuation({
        (continuation: CheckedContinuation<DetailedImage, Error>) in
        downloadImageAndMetadata(imageNumber: imageNumber) { image, error in
            if let image = image {
                continuation.resume(returning: image)
            } else {
                continuation.resume(throwing: error!)
            }
        }
    })
}
```

The magic behind this function occurs inside the `withCheckedThrowingContinuation` part. This function will give us a `CheckedContinuation<T, E> where E: Error` object that provides us with methods we need to call. In this example, the original version of `downloadImageWithMetadata` passes us a `DetailedImage` or an error, and we need to call the right `resume` method depending on what we get. If this method called us with a `Result<DetailedImage, Error>`, we could call `.resume(with:)` and pass it the `result` directly.

Continuations **must be called exactly once**, therefore there must be a continuation call within every branch of `withCheckedThrowingContinuation`. If you forget to call a `.resume`, things could go awry. Luckily, Swift will let you know.

**Note**: *Or least, it is supposed to. This article is based on the last few minutes of the [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/?time=1733) session. At least as of Beta 1, I was able to have code with branches that don't call `resume`.*

And just like that, we have converted closure-based code into something prettier! Using the `async/await` version of this function is as easy as:

```swift
Task {
    if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
        self.imageView.image = imageDetail.image
        self.metadata.text = "\(imageDetail.metadata.name) (\(imageDetail.metadata.firstAppearance) - \(imageDetail.metadata.year))"
    }
}
```

If you want to see and run a program using this, you can download a sample project from [here](/archives/CheckedContinuations.zip).

## Converting delegate-based code into async/await.

Up to now we have seen how you can convert callback-based code into async/await. You can also do this with delegate-based code. While delegate-based APIs have mostly disappeared in favor of callbacks, it is still common to encounter them, especially if the APIs in question are event-driven (Bluetooth, Location, etc). As such, you may benefit from knowing you can also bridge these to async/await.

Suppose you have an UIKit app that lets users choose contacts in a ViewController. In its simplest form, it may look similar to this:

```swift
class ViewController: UIViewController, CNContactPickerDelegate {

    @IBOutlet weak var contactNameLabel: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    @IBAction func chooseContactTouchUpInside(_ sender: Any) {
        showContactPicker()
    }

    func showContactPicker() {
        let picker = CNContactPickerViewController()
        picker.delegate = self
        present(picker, animated: true)
    }

    func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
        self.contactNameLabel.text = contact.givenName
        picker.dismiss(animated: true, completion: nil)
    }

}
```

Pressing a "choose contact" button will call `showContactPicker`, displaying the actual picker and, when the user selects the contact, the system will notify us of the event through the `contactPicker(_:contact)` method.

But we can do better. We can instead create an object that will wrap all this Contacts stuff for us. We can then create `async` methods that will let us know when a user has selected a contact. With this, we will be able to keep linearity in our program and keep a flow that is easier to follow.

We can declare `ContactPicker` as follows:

```swift
@MainActor
class ContactPicker: NSObject, CNContactPickerDelegate {
    private typealias ContactCheckedContinuation = CheckedContinuation<CNContact, Never>

    private unowned var viewController: UIViewController
    private var contactContinuation: ContactCheckedContinuation?
    private var picker: CNContactPickerViewController

    init(viewController: UIViewController) {
        self.viewController = viewController
        picker = CNContactPickerViewController()
        super.init()
        picker.delegate = self
    }

    func pickContact() async -> CNContact {
        viewController.present(picker, animated: true)
        return await withCheckedContinuation({ (continuation: ContactCheckedContinuation) in
            self.contactContinuation = continuation
        })
    }

    func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
        contactContinuation?.resume(returning: contact)
        contactContinuation = nil
        picker.dismiss(animated: true, completion: nil)
    }
}
```

What you need to understand here is:

1. We are typealiasing `CheckedContinuation<CNContact, Never>` so it's easier to refer to. Since we can't get an error, the error parameter is `Never`.
2. `private var contactContinuation: ContactCheckedContinuation?` will hold a reference to the continuation itself. This continuation is given to us in the `withCheckedContinuation` handler. It's an optional, because to avoid it from getting called more than once, we will set it to `nil` after the first call.
3. `pickContact` is `async`, as it will return the `CNContact` to us. We call `withCheckedContinuation` here.
4. When the contact is picked, we will call the continuation with `resume`.

And then, to use this:

```swift
@IBAction func chooseContactTouchUpInside(_ sender: Any) {
    async {
        let contactPicker = ContactPicker(viewController: self)
        let contact = await contactPicker.pickContact()
        self.contactNameLabel.text = contact.givenName
    }
}
```

But, note that our implementation has a flaw. If you have used the `ContactsUI` framework before, you may have caught it.

The UI presented gives our users the option to cancel without choosing a contact. Earlier we said that when dealing with continuations, you need to call the continuation exactly once. In the program above, we are not implementing the `contactPickerDidCancel(_)` method, and therefore our continuation is not getting called when users cancel.

To solve this, we have two options: We can throw an error when users cancel, or we can pass in a nil contact. It doesn't make much sense to throw an error in this case, so we will modify the code to take a nil contact instead.

```swift
class ContactPicker: NSObject, CNContactPickerDelegate {
    private typealias ContactCheckedContinuation = CheckedContinuation<CNContact?, Never>

    private unowned var viewController: UIViewController
    private var contactContinuation: ContactCheckedContinuation?
    private var picker: CNContactPickerViewController

    init(viewController: UIViewController) {
        self.viewController = viewController
        picker = CNContactPickerViewController()
        super.init()
        picker.delegate = self
    }

    func pickContact() async -> CNContact? {
        viewController.present(picker, animated: true)
        return await withCheckedContinuation({ (continuation: ContactCheckedContinuation) in
            self.contactContinuation = continuation
        })
    }

    func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
        contactContinuation?.resume(returning: contact)
        contactContinuation = nil
        picker.dismiss(animated: true, completion: nil)
    }

    func contactPickerDidCancel(_ picker: CNContactPickerViewController) {
        contactContinuation?.resume(returning: nil)
        contactContinuation = nil
    }
}

//...

// in ViewController

@IBAction func chooseContactTouchUpInside(_ sender: Any) {
    async {
        let contactPicker = ContactPicker(viewController: self)
        let contact = await contactPicker.pickContact()
        self.contactNameLabel.text = contact?.givenName
    }
}
```

This is much better. We will now call `resume` in all possible paths, our program will always be in a valid state, and, while we did write more code, there will be cases in which going the extra mile to preserve linearity will benefit the structure of the program in the long run. \*

You can download a full version of the contact picker app [here](/archives/AsyncAwaitContactPicker.zip). It's a UIKit app with a simple button and labels that shows you the given name of the contact you selected. Hopefully it will help you better understand the contents of this article.

# Summary

In this article we have explored how we can bridge from callback-based code or delegate-based code into `async/await`. We learned how to use checked continuations to do so, and we enforced the idea of what a continuation actually is.

With this, you should now know have all the essentials of `async/await`, You are now ready to tackle actual concurrency, and I'm happy to tell you will explore concurrency next week covering *structured concurrency*. You will learn how to run many tasks in parallel and how to process such results.

# Notes

\*: You should always stop and think if going that extra mile is actually worth it or if it is overkill. Over-engineering is a real and common problem in software engineering.
