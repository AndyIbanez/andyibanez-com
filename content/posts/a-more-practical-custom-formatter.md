---
title: "A More Practical Custom Formatter "
date: 2020-10-21T07:00:00-04:00
originalDate: 2020-10-20T11:15:09-04:00
publishDate: 2020-10-21T07:00:00-04:00
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
description: "Implement a more interesting custom Formatter in Swift."
keywords:
 - nsformatter
 - swift
 - programming
 - apple
 - ios
 - ipados
---

I had announced that I wouldn't be able to write an article this week due to it being Elections Day in my country. that said I just can't leave you guys without article, so this is a short one.

We will write another `NSFormatter` subclass. It will allow us to compose E-mails easily. This should also help show that formatters can format string into anything else really.

In short, we will be able to "compose" emails using raw strings, like this:

```text
TO: andy@andyibanez.com
FROM: andy.cito@gmail.com
CC: gg@hotmail.com, rk@hotmail.com
BCC: myboss@gmail.com,myarchitect@gmail.com
SUBJECT: Important Notice
BODY:
Hello guys. Just wanted to let you know that this is a very important notice. The notice has been sent and it's importance has priority one.

Please take note of the importance, and share it with everyone you need to.
```

We will be able to convert that into this:

```swift
class Email {
    let to: String?
    let from: String?
    let subject: String?
    let cc: [String]?
    let bcc: [String]?
    let body: String?
    
    init(
        to: String? = nil,
        from: String? = nil,
        subject: String? = nil,
        cc: [String]? = nil,
        bcc: [String]? = nil,
        body: String? = nil) {
        self.to = to
        self.from = from
        self.subject = subject
        self.cc = cc
        self.bcc = bcc
        self.body = body
    }
}
```

And viceversa.

The only constraint will be that the `BODY:` always has to go last. Any fields can be missing and they will be filled as nil when appropriate.

# The EmailFormatter Class

The EmailFormatter will format plain text into emails and the other way around. It will inherit from `Formatter`.

## Overriding string(for:)

We will start by implementing `override func string(for obj: Any?)`, as it is the easiest case. This object will ultimately take an `Email` and return the string representation of it.

```swift
override func string(for obj: Any?) -> String? {
    guard let email = obj as? Email else { return nil }
    var stringRep = ""
    if let to = email.to {
        stringRep += "TO: \(to)\n"
    }
    if let from = email.from {
        stringRep += "FROM: \(from)\n"
    }
    if let cc = email.cc {
        let joined = cc.joined(separator: ",")
        stringRep += "CC: \(joined)\n"
    }
    if let bcc = email.bcc {
        let joined = bcc.joined(separator: ",")
        stringRep += "BCC: \(joined)\n"
    }
    if let subject = email.subject {
        stringRep += "SUBJECT: \(subject)\n"
    }
    if let body = email.body {
        stringRep += "BODY:\n\(body)"
    }
    return stringRep
}
```

We are also going to expose a prettier signature for our string formatter.

```swift
public func stringFor(_ email: Email) -> String? {
    return string(for: email)
}
```

# Converting strings into Emails

The hardest part is now to convert strings into our object. In this specific case, it isn't too complicated, but know that it can become hairy quickly.

```swift
override func getObjectValue(
    _ obj: AutoreleasingUnsafeMutablePointer<AnyObject?>?,
    for string: String,
    errorDescription error: AutoreleasingUnsafeMutablePointer<NSString?>?) -> Bool {
    obj?.pointee = stringToEmail(emailString: string)
    return true
}
```

`stringToEmail` is implemented as such (can definitely be improved):

```swift
func stringToEmail(emailString: String) -> Email {
    let splat = emailString.split(separator: "\n")
    var to: String? = nil
    var from: String? = nil
    var subject: String? = nil
    var cc: [String]? = nil
    var bcc: [String]? = nil
    var body: String? = nil
    
    for line in splat {
        if line.hasPrefix("TO: ") {
            let toLine = line.split(separator: ":")
            to = { if let to = toLine.last { return String(to) } else { return nil }}()
        }
        
        if line.hasPrefix("FROM: ") {
            let fromLine = line.split(separator: ":")
            from = { if let from = fromLine.last { return String(from) } else { return nil }}()
        }
        
        if line.hasPrefix("SUBJECT: ") {
            let subjectLine = line.split(separator: ":")
            subject = { if let subject = subjectLine.last { return String(subject) } else { return nil }}()
        }
        
        if line.hasPrefix("CC: ") {
            let ccLines = line.split(separator: ":").last?.split(separator: ",").map { String($0) }
            cc = ccLines
        }
        if line.hasPrefix("BCC: ") {
            let bccLines = line.split(separator: ":").last?.split(separator: ",").map { String($0) }
            bcc = bccLines
        }
    }
    
    let bodyStart = splat.firstIndex { $0.hasPrefix("BODY: ") }
    let nextIndex = (bodyStart ?? 0) + 1
    if nextIndex < splat.count {
        let linesToMerge = nextIndex..<splat.count
        body = linesToMerge.reduce(""){ "\($0 ?? "")\n\(splat[$1])" }
    }
    
    return Email(
        to: to,
        from: from,
        subject: subject,
        cc: cc,
        bcc: bcc,
        body: body)
}
```

And with that, we have a formatter that can convert from and to strings and `Email` objects.

Sample usage is below:

```swift
let email = Email(to: "me@andyibanez.com", bcc: ["gg@hotmail.com", "rk@yahoo.es"])

let formatter = EmailFormatter()
let string = formatter.stringFor(email)

print(string!)

let emailString =
"""
TO: andy@andyibanez.com
FROM: andy.cito@gmail.com
CC: gg@hotmail.com, rk@hotmail.com
BCC: myboss@gmail.com,myarchitect@gmail.com
SUBJECT: Important Notice
BODY:
Hello guys. Just wanted to let you know that this is a very important notice. The notice has been sent and it's importance has priority one.

Please take note of the importance, and share it with everyone you need to.
"""

let newEmail = formatter.email(from: emailString)!

print("We will send an email to \(newEmail.to!) and \(newEmail.cc!.count) others")
```

# Conclusion

Hopefully this article shows you a bit better everything you can do with your custom formatters. Having the flexibility to convert anything into string and back is great, and there's support for it all over the frameworks.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.