---
title: "Common Cryptographic Operations With Cryptokit"
date: 2019-10-09T07:00:00-04:00
originalDate: 2019-10-05T18:08:07-04:00
draft: false
publishDate: 2019-10-09T07:00:00-04:00
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
description: "Learn to implement basic cryptography with CryptoKit in Swift."
keywords:
 - swift
 - cryptokit
 - ios
 - tvos
 - ipados
 - watchos
 - wwdc2019
---

Apple has always taken security very seriously, so it's expected that they would provide developers with the same tools they have to help developers implement the same security measures in their apps. This year, Apple introduced `CryptoKit`.

Apple providing new cryptography tools is nothing new. They have provided the `Security` framework for a *very* long time, and a few years later they introduced `CommonCrypto`. The problem with these frameworks is that they can be very low level, being written in C, and it can be intimidating for new developers to adopt them in their project. `CryptoKit` abstracts a lot of the details and it provides easier interfaces for common operations such as hashing, encrypting, and even signing.

In this article we will explore how to do common cryptographic operations with CryptoKit, and the downsides it currently has.

# Introduction to CryptoKit

`CryptoKit` is a cryptography framework for Apple's platforms written in Swift. It provides easy and convenient interfaces for cryptographic operations in a safe and high-level manner. You no longer have to worry about managing pointers or other low-level concepts that just don't exist in Swift. You also don't have to do manual memory management.

CryptoKit allows you to:

* Compute and compare hashes.
* Work with Public-Key cryptography to create and evaluate digital signatures and do key exchange.
* Work with symmetric cryptography to do message authentication and encryption.

# Common Cryptographic Operations with CryptoKit

## Hashing

To perform hashing, `CryptoKit` provides the [`HashFunction`](https://developer.apple.com/documentation/cryptokit/hashfunction) protocol, along with three implementations of it. At the time of this writing, said implementations are the following structs:

* SHA256
* SHA384
* SHA512

They are all used similarly. They all have a static method that takes a `DataProtocol` of what you want to hash.

For example, to perform a SHA256:

```swift
let data = string.data(using: .utf8)!
let hash = SHA256.hash(data: data)
```

This will return the hash as a `SHA256.Digest` object, which is actually a collection

If you want to get the string representation of this hash, you do this:

```swift
let stringHash = hash.map { String(format: "%02hhx", $0) }.joined()
```

<hr>

**Important Note!**

You may have seen (and maybe even been tempted by) people grabbing the hash using the `description` property of the digest.

```swift
print(hash.description)
```

This is because the property prints something like this:

```text
SHA256 digest: ae7ffb85a76d5810c70c2459415e02b26a556e6d02ef76449690c1459232ffa9
```

This is not safe, because Apple can change how the `description` property returns the information about the hash in a future release. In general, you should never rely on this property to get any long-lasting data. You should only use it when debugging.

As a recent example of this, Apple recently changed what the `description` property of `Data` objects returns. Developers were using it as a quick way to get the textual representation of a push notification token, and Apple changed it to return the size of the data instead. This caused a lot of broken push notifications in a lot of apps.

<hr>

For the sake of comparison, this is how I calculated a `SHA512` hash prior to iOS 13 in Swift:

```swift
  func iOS10Sha512(data: Data) -> Data {
    let digest = NSMutableData(length: Int(CC_SHA512_DIGEST_LENGTH))!
    let value = data as NSData
    let uint8Pointer = UnsafeMutablePointer<UInt8>.allocate(capacity: digest.length)
    CC_SHA512(value.bytes, CC_LONG(data.count), uint8Pointer)
    return value as Data
  }
```

All the aforementioned hashes are used the same way. If you need to use deprecated hash algorithms but are still popular today (such as `MD5`), Apple provides them in an `Insecure` enum.

To calculate the MD5 hash of a piece of data with CryptoKit:

```swift
let md5 = Insecure.MD5.hash(data: data)
```

You can also find the `SHA1` algorithm within `Insecure`.

## Symmetric Encryption

With CryptoKit, when you encrypt a piece of data not only will get you confidentiality of your message, but also authenticity.

CryptoKit provides two symmetric cyphers: `AES-GCM` and `ChaChaPoly`.

Their usage is once again very straightforward and you don't have to concern yourself with the low-level implementation details.

The return type of the `seal` method is a `AES.GCM.SealedBox` object which contains information about the box. A few important properties:

* A `ciphertext`, which is the encrypted data with the same size as the input data.
* A `tag`, which ensures the cannot be tampered with in a way you would not notice.
* A `nonce`, which is a random number to add entropy to the encrypted data.

If you need to share the data with somebody else, you can use the `combined` property which combines all the previous properties into one. Then they can decrypt this data using the same key.

```swift
let combinedData = sealedBox.combined! // Previous sealed box
let sealedBoxToOpen = try! AES.GCM.SealedBox(combined: combinedData)
let decryptedData = try! AES.GCM.open(sealedBoxToOpen, using: passwordKey)
let decryptedString = String(data: decryptedData, encoding: .utf8)!
print(decryptedString) // "The Legend of Zelda"
```

## Signing and Verifying Signed Content

CryptoKit can help you sign and verify the signature of data.

CryptoKit comes with four different elliptic curve types:

* `Curve25519`
* `P521`
* `P384`
* `P256`

We will see how to use P521, and you will be able to use the others with these examples.

### Generating Key Pairs

Public-Key cryptography works with private and public keys, so we will start generating those.

```
let privateKey = P521.Signing.PrivateKey()
let publicKey = privateKey.publicKey
let publicKeyData = publicKey.rawRepresentation //You can share this one with others
```

The `P521.Signing.PrivateKey` object wraps both the private key and public key, and they both have a `Data` representation that you can use to share (though you shouldn't share the private key with anyone, but you can use the data representation if you want to store it in a different way). You can convert these representations to Base64 encoded strings to make them easier to share, like posting them on your websites or sharing them in social media.

### Signing

Dealing with signing is much, much easier than doing it with the earlier APIs (and I tell you this from personal experience!)

#### Creating Signatures

Signing is a very simple affair, and so is verifying signatures.

```swift
let signature = try! privateKey.signature(for: data)
```

There's two variations of the `signature` method: One for data, and one for digests, which is what you would normally want to do. If you have a digest generated with CryptoKit's `SHA256`, `SHA384`, `SHA512`, or even an insecure algorithm, you can sign it directly with the overloaded method.

```swift
let hash = SHA256.hash(data: data)
let digestSignature = try! privateKey.signature(for: hash)
```

#### Verifying Signatures

Just like is the case with signing, there's two overloaded methods for signature verification: One for a `DataProtocol`, and another one for a digest.

```swift
if publicKey.isValidSignature(signature, for: data) {
  print("Valid signature")
}
```

<hr>
**Important Note!**

Not all cyphers have signing methods for both digests and data. Curve25519 only exposes signing methods for `DataProtocol`s.
<hr>

### Key Agreement

Key agreement is a method used for multiple parties to securely choose a shared encryption key that can be used for signing and encrypting the data they want to share between each other.

Start by choosing a random salt.

```swift
func randomData(length: Int) -> Data {
  var data = Data(count: length)
  _ = data.withUnsafeMutableBytes {
    SecRandomCopyBytes(kSecRandomDefault, length, $0.baseAddress!)
  }
  return data
}

let salt = randomData(length: 32) // 256bits
```

If you don't want to drop down to CommonCrypto to generate a random salt, you can create a new `SymmetricKey` of 256bits, and than grab its raw bytes to use it as a salt.

```swift
let symKeySalt = SymmetricKey(size: .bits256)
let salt = symKeySalt.withUnsafeBytes { Data($0) }
```

After a salt has been generated, all the interested parties need to share their public keys with each other.

```swift
let alicePrivateKey = P521.KeyAgreement.PrivateKey()
let alicePublicKey = alicePrivateKey.publicKey

let eileenPrivateKey = P521.KeyAgreement.PrivateKey()
let eileenPublicKey = eileenPrivateKey.publicKey

/// Alice sends her public key to Eileen
/// Eileen sends her public key to Alice
```

Now, all the parties have the relevant public keys.

Then, all the parties need to do is derive a secret using their own private key and the public key of the people they want to talk with. Once they have that shared secret, they can get the symmetric key.

All parties should derive the same symmetric key, and later they can start sharing data encrypted using this key.

```swift
// Alice derives the shared secret and key.
let aliceSharedSecret = try! alicePrivateKey.sharedSecretFromKeyAgreement(with: eileenPublicKey)
let aliceSymmetricKey = aliceSharedSecret.hkdfDerivedSymmetricKey(using: SHA256.self, salt: salt, sharedInfo: Data(), outputByteCount: 32)

// Eileen derives the shared secret and key.
let eileenSharedSecret = try! eileenPrivateKey.sharedSecretFromKeyAgreement(with: alicePublicKey)
let eileenSymmetricKey = eileenSharedSecret.hkdfDerivedSymmetricKey(using: SHA256.self, salt: salt, sharedInfo: Data(), outputByteCount: 32)

if aliceSymmetricKey == eileenSymmetricKey {
  print("The keys are equal and now they can share data")
}
```

And said encryption can be done with the two algorithms we saw before: Either AES-GCM or ChaCha20. To show this we will simulate a simple chat between Alice and Eileen:

```swift
// Alice sends a message to Eileen
let message = "Hi Eileen!".data(using: .utf8)!
let encryptedByAlice = try ChaChaPoly.seal(message, using: aliceSymmetricKey)

 // Eileen reads Alice's message.
let decryptedMessage = try! ChaChaPoly.open(encryptedByAlice, using: eileenSymmetricKey)
let decryptedMessageString = String(data: decryptedMessage, encoding: .utf8)!
print(decryptedMessageString) // Hi Eileen!
```

You can send a different salt each time a new message is generated as to keep it unique, and I encourage you to do so. If you are starting with CryptoKit, feel free to use a hard coded salt, but don't use it in a real application.

# The Downsides

Currently, CryptoKit doesn't have much support for some popular encryption algorithms. While I wouldn't expect Apple to implement TwoFish or Serpent anytime soon, I was surprised when I saw we couldn't derive RSA keys. CryptoKit is really nice though, so I'm sure the encryption suites it supports will just grow in the future.

# Conclusion

CryptoKit is a new and modern Cryptography framework for Apple's platforms. It is very high level, which makes it very easy to use. It supports a good set of cryptography algorithms you'd expect to find in any other library. It supports the most basic operations like hashing, encryption, and even key derivation and sharing. It's a very powerful and simple framework despite it's lack of other popular algorithms. I personally have high hopes for its future, and it was one of my favorite surprises of this year's WWDC.

<hr>
**Important Note!**

In this article, I took a lot of liberties with the force-unwrapping and `try!`. With CryptoKit you should always deal with these operations in a safe manner.
<hr>

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.