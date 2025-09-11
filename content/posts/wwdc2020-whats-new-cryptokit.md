---
title: "WWDC2020: What's new in CryptoKit"
date: 2020-09-14T07:00:00-04:00
originalDate: 2020-09-14T07:00:00-04:00
draft: false
publishDate: 2020-09-14T07:00:00-04:00
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
 - wwdc2020
categories:
 - development
description: "Learn about the new features Apple added to CryptoKit in WWDC2020."
keywords:
 - swift
 - cryptokit
 - ios
 - tvos
 - ipados
 - watchos
 - wwdc2020
---

CryptoKit, introduced in WWDC2019, allows us to [perform cryptographic operations very easily](https://www.andyibanez.com/posts/common-cryptographic-operations-with-cryptokit/). 

While CryptoKit still doesn't offer many algorithms and functionality, it's still growing, and this year CryptoKit and do more.

# HKDF

Key derivation functions have been available from day one, but it wasn't possible to derive keys independently. It was only possible to do so if you were using elliptic curve key agreement protocols.

To do this, there is a new `HKDF` object with static methods. One such method is `deriveKey` with multiple overloads:

```swift
let key = SymmetricKey(size: .bits256)
let info = "abcdef".data(using: .utf8)! // Quick example only
let derivedKey = HKDF<SHA512>.deriveKey(inputKeyMaterial: key, info: randomInfo, outputByteCount: 256)
print(derivedKey.bitCount)
```

Both parties would need to have same info in order to derive the same keys. This is because this is a deterministic algorithm and the same inputs will always produce the same outputs.

# Creating Elliptic Curve Keys with PEM and DER encoded strings.

Something that I was really missing in CryptoKit was the ability to store EC keys in a standard format. Starting on CryptoKit 1.1.0 (iOS 14 and the other OSes announced at WWDC2020), we can create our EC keys with PEM and DER encoded strings.

To support this, all Private and Public keys inside EC wrappers now have two new initializers - `init(pemRepresentation:) throws` and `init(derRepresentation:) throws`.

```swift
let pemKey = """
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIKd4xNZ5A90r8jDUkfu9MrTscvKia9ebl2SDoPttK1C9oAoGCCqGSM49
AwEHoUQDQgAEdF+4auFmiRknxAXXb7X0QMfp6l/JGpf+2FUNkPaIBAODztGa6XNQ
ItQzQGNo26D3DCe8LL/vQpMnNX1ONL6Ocw==
-----END EC PRIVATE KEY-----
"""

let p256FromPem = try P256.KeyAgreement.PrivateKey(pemRepresentation: pemKey)
```

The `pemKey` is a string. If you want to play around with this, I generated the `pemKey` using OpenSSL on my Mac using the following command in the terminal:

```
openssl ecparam -genkey -name prime256v1 -noout -out ec256-key-pair.pem
```

Needless to say, don't use this private key for anything as it has been posted publicly.

And not only can you create keys based on their PEM and DER representations, you can also get these representations from brand new keys created directly with CryptoKit.

```swift
let cryptoKitP521KeyPair = P521.KeyAgreement.PrivateKey()
let cryptoKitP521PublicKey = cryptoKitP521KeyPair.publicKey

print("PRIVATE KEY:\n\n\(cryptoKitP521KeyPair.pemRepresentation)")
print("PUBLIC KEY:\n\n\(cryptoKitP521PublicKey.pemRepresentation)")
```

```
PRIVATE KEY:

-----BEGIN PRIVATE KEY-----
MIHuAgEAMBAGByqGSM49AgEGBSuBBAAjBIHWMIHTAgEBBEIBuL2ZAOozjAd+SS54
ipH72btEIRcuWmzHZE2d+9fe+iMCfDnIT+/XF7+rUboSBvlQqFX/X/S96ddvbNdc
bx3Ii3GhgYkDgYYABACm8IjMS9Ql0/xm8HtaJCalqceBGP1ydltl257TZB9O92zi
LTwlzyceyMTQ6wdiY58BYkhs9WybldbGfV6OI8Jm1wAE3q+Gum/2bgf2ZVeU50gD
h/N8Kpj2F8HVhph4aPci/ixZ84DfyIzU/8OxApGLW/0Ixdxy7XaUTiwwpBVIgvW4
Lw==
-----END PRIVATE KEY-----
PUBLIC KEY:

-----BEGIN PUBLIC KEY-----
MIGbMBAGByqGSM49AgEGBSuBBAAjA4GGAAQApvCIzEvUJdP8ZvB7WiQmpanHgRj9
cnZbZdue02QfTvds4i08Jc8nHsjE0OsHYmOfAWJIbPVsm5XWxn1ejiPCZtcABN6v
hrpv9m4H9mVXlOdIA4fzfCqY9hfB1YaYeGj3Iv4sWfOA38iM1P/DsQKRi1v9CMXc
cu12lE4sMKQVSIL1uC8=
-----END PUBLIC KEY-----
```

We can now store our keys in standard formats to be be able to easily use them in other apps.

# Conclusion

CryptoKit keeps improving. This year we got very nice features that should help with both the interoperability of CryptoKit with other systems and basic HKDF.

