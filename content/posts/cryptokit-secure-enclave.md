---
title: "CryptoKit and the Secure Enclave"
date: 2020-01-15T07:00:00-04:00
originalDate: 2020-01-13T21:18:51-04:00
draft: false
publishDate: 2020-01-15T07:00:00-04:00
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
 - cryptokit
 - wwdc2019
categories:
 - development
description: "Learn about the limitations of CryptoKit, and how to go around them."
keywords:
 - swift
 - cryptokit
 - ios
 - tvos
 - ipados
 - watchos
 - wwdc2019
---

CryptoKit and the Secure Enclave

Apple's [CryptoKit](https://www.andyibanez.com/posts/common-cryptographic-operations-with-cryptokit/) introduced this year is full of amazing features. Not only does it offer very easy to use cryptography, but it also offers an interface to a security feature that Apple introduced less than a decade ago: The Secure Enclave.

The Secure Enclave is a hardware feature for helping the system work with cryptographically secure data. In this article, we will build upon our previous CryptoKit knowledge (see the article linked above), and we will also learn what the Secure Enclave is all about.

# The Secure Enclave

Before we dive into writing some code, we need to understand what the Secure Enclave actually *is*. For some, it is nothing more than a buzzword from the past few years. But for developers and security-conscious people, it is a hardware feature for cryptographic tasks.

The Secure Enclave first appeared on the Apple A7 chip, released on the iPhone 5S all the way back to 2013. Ever since then, all new iOS and iPadOS devices support it, and it's even available in MacBooks with a TouchBar. The iPhone 5S is also the first Apple device to support biometric authentication. Coincidence?

Secure Enclave and Touch ID/Face ID go hand in hand. When you enroll your fingerprint or face for biometric unlock on your device, a mathematical representation of it is stored on the Secure Enclave - an actual photo of your face or your fingerprints themselves are _never_ stored in the Secure Enclave. Your device passcode is also handled by the Secure Enclave.

The Secure Enclave runs its own operating system separate from iOS, and iOS never has direct access to the data in the Secure Enclave, or the other way around. Instead, when iOS needs anything from it, it queries it with a question and the Secure Enclave responds with an answer. In other words, when you unlock your phone, iOS takes a reading of your face, generates mathematical representation of it, and it asks the Secure Enclave "does this face data matches the one you have?" - If the Secure Enclave responds "yes", then the phone is unlocked. If it isn't, you see an error. The Secure Enclave does not return the face data to iOS, so iOS itself cannot do the checking. The data in the Secure Enclave is truly secure and it never leaves its cozy place.

For us developers, how the Secure Enclave deals with biometrics is not the most exciting part about it, because we cannot query it directly. Even our biometric APIs are constrained and they are fully handled by the system, so we cannot really do much work on top of that. The real exciting thing is that we as developers can leverage the Secure Enclave to encrypt and decrypt information with keys that are specific to a specific setup in a specific device.

We can create asymmetric keys directly on the Secure Enclave with both the old `Security` framework, or through CryptoKit. We cannot create a cryptographic key on some other platform or non-Secure Enclave piece of code and import it into the Secure Enclave later. But we can use CryptoKit to generate a key pair whose private key is stored on the Secure Enclave. This means that we can encrypt data that is only accessible via the device that encrypted it. Because keys created directly on the Secure Enclave cannot be retrieved, you cannot decrypt the info on another device. Moreover, when I say that the keys are specific to that specific *setup* and device, I mean that the key is specific that installation of iOS. If you do a clean install of your phone, the Secure Enclave is wiped clean. If your users restore a version of your app that had information encrypted by the secure enclave on another device than the original one, that information also becomes inaccessible.

At the time of this writing, the Secure Enclave offers 4MB of Flash Storage for keys, and it only supports P256 elliptic curve keys. That said, 4MBs of storage is plenty of storage for a bunch of keys.

With all that said, CryptoKit can ask the Secure Enclave to sign data, and to perform key agreement with a set of keys. The API you already know from the previous article doesn't change much when dealing with the Secure Enclave.

# CryptoKit and the Secure Enclave.

<hr>
**Important Note!**

The Secure Enclave is only available on physical iOS devices with the A7 chip and later. As such, the code here will not work on the simulator. You need to run it on an iPhone 5S or another iOS/iPadOS device that came after it.
<hr>

To actually interact with the Secure Enclave, CryptoKit offers the `SecureEnclave` enum. The first thing you may want to do is to check if the Secure Enclave is available in the device your app is running:

```swift
if SecureEnclave.isAvailable {
  // Secure Enclave is available.
}
```

Once you know the Secure Enclave is available, you can start using it. I find it helpful to think of iOS and the Secure Enclave as two different parties who want to share secret information with each other. To do this, each party needs their own public key pair, and then they need to exchange their public keys. Then when they send information to each other, they need to sign it with their private keys and generate a shared secret with each other's public key.

The below example will generate two key pairs: One within iOS (`nonEnclaveKeys`) and one within the Secure Enclave `enclaveKeys`. As we said before, the enclave only supports P256 keys for now, so we will use that.

```swift
let enclaveKeys = try! SecureEnclave.P256.KeyAgreement.PrivateKey()
```

You can get the data representation of the private key file and store it in the keychain, or even on a plain file, if you so decide. After all, you need to persist the key if you want to use it again.

```
let dataRepresentation = enclaveKeys.dataRepresentation
```

We are not going to explore all the cryptographic operations you can do with the SecureEnclave - The way you utilize the `SecureEnclave` is exactly the same as the other [common cryptographic operations](https://www.andyibanez.com/posts/common-cryptographic-operations-with-cryptokit/) without using the Enclave, but we have a few other things to discuss.

## The Data Representation of A Private Key.

If you have been paying attention, you may be confused at the fact that you can get a data representation of the private key, which is supposed to be stored on the Secure Enclave. Aren't keys stored on the Secure Enclave supposed to be entirely inaccessible by the device? Why can we get a data representation of a private key stored on the Secure Enclave? What's going on here?

If you think about it, you *do* need a way to get a private key from the Secure Enclave. Otherwise, you would create thousands of one-time keys (surprisingly, was not able to find a way to delete Secure Enclave keys with CryptoKit), and you'd need to share a new public key every time you want to share encrypted data with someone.

It turns out that, keys you store in the Secure Enclave have a data representation as well as other CryptoKit keys, but in the case of the Secure Enclave keys, they are not the raw keys. Rather, the Secure Enclave gives you an encrypted block that only the Secure Enclave itself can later use to restore the real key. So it all works as you would expect: We have no access to the raw keys on the Secure Enclave, but just a representation of it.

You don't have to take my word for it. The [documentation](https://developer.apple.com/documentation/cryptokit/storing_cryptokit_keys_in_the_keychain) states:

> Keys that you store in the Secure Enclave expose a raw representation as well, but in this case the data isn’t the raw key. Instead, the Secure Enclave exports an encrypted block that only the same Secure Enclave can later use to restore the key.

If you have two devices, you can do a little experiment to verify that the rawRepresentation of a Secure Enclave key generated on one device cannot be used on another device.

Generate a key pair, save it to disk:

```swift
try! nonEnclaveKeys.rawRepresentation.write(to: pathToUrl)
```

And on the other device, import this key (you can just enable iTunes File Sharing on a test app), and open it:

```swift
let keyData = try! Data(contentsOf: pathToFile)
let nonEnclavekeys = P256.KeyAgreement.PrivateKey(rawRepresentation: keyData)
```

If you print the data in both the original device and the other device, you will see it is the same. CryptoKit will also have no issue "loading" the key on the other device.

But, the moment you try to use the key on the other device, you will see that it doesn't work. That's because the Secure Enclave on the new device is trying to open the key with its internal keys, and because the encrypted blob was made by another device, it cannot do that.

Using CryptoKit with the Secure Enclave ensures that encrypted data created on a device can only be decrypted by the same device, on the same setup. If you do a restore of the device, the data will no longer be accessible.

## CryptoKit, SecureEnclave, and Local Authentication

There is one specific API that is not available to non-Secure Enclave keys: We can make private key representations available only after an user has authenticated themselves (with `LAContext`) and the conditions under which a key will be available (`SecAccessControl`). For that, `SecureEnclave.P256.KeyAgreement.PrivateKey` has the following initializer:

```swift
init(compactRepresentable:accessControl:authenticationContext:)
```

If you are not familiar with these security APIs, `LAContext` allows you to ask the device to authenticate the device with biometrics (Touch ID or Face ID) or with their password, and `SecAccessControl` allows you to specify an access control to instruct the system under which conditions should a key be available. These conditions can be that the user should be authenticated first, the app needs a specific password, in a specific application password, and more. The usage looks like this:

```
import LocalAuthentication // For LAContext
import Security // For SecAccessControl

// ...

let authContext = LAContext();
    
let accessControl = SecAccessControlCreateWithFlags(
   kCFAllocatorDefault,
   kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
   [.privateKeyUsage, .userPresence, .biometryCurrentSet],
   nil
)!;
    
let privateKey = try! SecureEnclave.P256.KeyAgreement.PrivateKey(
  accessControl: accessControl,
  authenticationContext: authContext)
```

We are creating a private key that will only be available after a user has been authenticated. LAContext will take care of doing the authentication itself.

This will return you another raw representation of the private key you can store. To load a key created with conditions, you can use the `init(dataRepresentation:authenticationContext:)` initializer:

```swift
try! CryptoKit.SecureEnclave.P256.KeyAgreement.PrivateKey(dataRepresentation: keyData, authenticationContext: context)
```

You only need the context to perform the authentication. The access control is only created at the time you create the key and the key persist under those conditions.

# Conclusion

In this article, we learned a little bit more CryptoKit by exploring how it can integrate with the Secure Enclave. The Secure Enclave is an area that stores secret information and no information can ever leave it. We also learned how we can use other device features - such as Touch ID and Face ID - to further secure our keys within the Secure Enclave.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.