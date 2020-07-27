---
title: "Adding Custom SwiftUI Views and Modifiers to the Xcode Library"
date: 2020-07-29T07:00:00-04:00
publishDate: 2020-07-29T07:00:00-04:00
originalDate: 2020-07-26T23:32:06-04:00
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
 - tvos
 - swiftui
 - xcode12
categories:
 - development
description: "Learn how to add custom views and modifiers to the Xcode library using the LibraryContentProvider protocol."
keywords:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - tvos
 - swiftui
 - xcode12
---

Xcode 12 introduces the ability for developers to add their own SwiftUI views and modifiers to the Xcode library. This allows people using your code to discover your custom views, makes your codebase easier to learn, and it allows you to visually edit complex views visually rather than with code.

In this short article we will explore how we can add our own views and modifiers to the Xcode library.

# The LibraryContentProvider Protocol

The fun thing about doing this is that it is done entirely in code. You do not have to mess with settings panes or anything of the sort in order to create and add your own SwiftUI content to the Xcode Library. Instead, all you need to do is to conform to the `LibraryContentProvider` protocol. This has the advantage that your custom views in the Library are per-project and the Library won't be polluted with possibly irrelevant views.

The protocol looks like this:

```swift
struct LibraryViewContent: LibraryContentProvider {
    @LibraryContentBuilder
    var views: [LibraryItem]
    
    @LibraryContentBuilder
    func modifiers(base: ModifierBase) -> [LibraryItem]
}
```

Simple enough - We have an array of views, and a function to return an array of modifiers, both of the type `[LibraryItem]`.

Xcode scans your code for objects conforming to this protocol and adds them to the library.

## The LibraryItem Object

The `LibraryItem` object, at the very least, expects the view that it will represent in the XcodeLibrary.

```swift
LibraryItem(MyView())
```

But we can provide it with more information:

```swift
LibraryItem(
	MyView(),
	visibility: true, // Whether the item should be visible in the library
	title: "My View", // An optional title for the view.
	category: .control, // The category this view belongs to
)
```

`category` is a `LibraryItem.Category` enum. It can be `.control`, `.effect`, `.layout`, and `.other`.

## Conforming to LibraryContentProvider

### LibraryContentProvider For Views

We are going to add this rounded rectangle view as an example:

```swift
struct RoundedTextView: View {
    var text: String
    var body: some View {
        Text(text)
            .padding(10)
            .background(Color.blue)
            .foregroundColor(.white)
            .clipShape(RoundedRectangle(cornerRadius: 30, style: .continuous))
    }
}
```

It looks like this:

![Rounded rect text view](/img/round_rect_library_item.png)

And that's it for views! Now when you open the Library (Tapping the `+` button, or pressing `Shift + Command + L`), you can search for your view and drag it and click it like you would any other view:

![Custom view in Library](/img/library_view_round_rect.png)

By default, you can see it's in a section called `customviewtexts`. This is group is the name of your project. You cannot really get rid of it, but if you specify another category such as `.control` you can have better control over the grouping:

![Custom View with Another Category](/img/custom_library_category.png)

You can add the same view as many times as you want but with different configurations. In our example, you could create a view with a different text and add that as a different item in the library. If you make properties such as the foreground and background colors, you could add them as different configurations as well.

### LibraryContentProvider for Modifiers

To add custom modifiers, start by creating your custom modifier on the view it applies to. In this example we will make a modifier for any view, but you can be as specific as you want:

```swift
extension View {
    func shadowAndSaturation(saturation: Double) -> some View {
        self
            .shadow(color: Color.red, radius: 30)
            .saturation(saturation)
    }
}
```

Now, we need to implement the `modifiers` function of `LibraryContentProvider`.

```swift
@LibraryContentBuilder
func modifiers(base: ModifierBase) -> [LibraryItem] {
}
```

The `ModifierBase` is the type of view our modifier should apply to. If you created your modifier in an extension for `Text`, then you would use text; if you used `Image`, you would specify `Image`. In our case it applies to any view, so we will leave it as `AnyView`.

And done! If you bring up the Library pane now, you can now see your custom modifier in the Modifiers section:

![Custom Modifier in Library](/img/custom_modifier_library.png)

# Final Discussion

Another neat thing about using this system is that you don't have to build and run your project in order for them to appear in the library. Xcode automatically scan and adds `LibraryContentProvider`s without your intervention. Even when your code is not in a runnable state, Xcode can scan and add custom views to its library.

When compiling your code for distribution, all `LibraryContentProvider`s are stripped from your code. And you can use them in Swift Packages as well.

# Conclusion

Being able to add your own views and modifiers to the library is a great feature. They are project-specific and you can add them with the SPM. Consider using this feature if you have a lot of reusable custom views.

<br>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.