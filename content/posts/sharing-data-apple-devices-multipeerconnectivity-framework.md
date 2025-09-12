---
title: "Sharing Data Across Apple Devices with the MultipeerConnectivity Framework"
date: 2020-07-15T07:00:00-04:00
publishDate: 2020-07-15T07:00:00-04:00
originalDate: 2020-07-12T21:48:33-04:00
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
 - multipeerconnectivity
categories:
 - development
description: "How to use the Speech framework to detect speech on iOS."
keywords:
 - swift
 - ios
 - ipados
 - tvos
 - multipeerconnectivity
---

There are times when we may want to share data across instances of our app running on different physical devices. You could develop a server or even leverage cloud storage, but did you know Apple provides a framework to share data *directly* across devices, without having to use an intermediary? This framework is the MultipeerConnectivity framework, and it has actually been around for a while. In this article, we will explore this framework to understand how we can use it to share data across instances of our app in different devices directly.

# The MultipeerConnectivity Framework.

This framework is actually very old. It was introduced in iOS 7 all the way back in 2013. macOS later supported in OS X 10.10 Yosemite in 2014, and it's even supported by tvOS starting on tvOS 10.0. It supports a wide arrange of devices.

The framework allows you to send basically any kind of data, whether it is short strings of text or images.

## How It Works

Before we dive deep into the code, we need to understand, superficially, the technologies it uses under the hood. This is important because you will be able to understand the capabilities and limitations of the framework in case you ever come across code that you expect to work, but doesn't.

What's important to know is that the framework can use different mediums to share data and not all devices support the same mediums.

### iOS Support

In iOS, the framework can use the following as the underlying medium for data sharing:

* Infrastructure Wi-Fi (AKA the Wi-Fi you have in your house).
* Peer-to-peer Wi-Fi.
* Bluetooth Personal Area Network (PANs).

### macOS and tvOS

tvOS and macOS both support the same transport mechanisms:

* Infrastructure Wi-Fi
* Peer-to-peer Wi-Fi
* Ethernet

## Overall Architecture

Devices cannot connect and send data to *any* device willy-nilly. Before two devices can share data, they need to establish a Session (a [`MCSession`](https://developer.apple.com/documentation/multipeerconnectivity/mcsession) object) with each other.

To do this, one of the device becomes the advertiser and it starts broadcasting to nearby devices. It simply tells them "hey all, I am willing to connect to one, as long as you guys are offering a session of this type. This is done with the [`MCNearbyServiceAdvertiser`](https://developer.apple.com/documentation/multipeerconnectivity/mcnearbyserviceadvertiser), object, or with the [`MCAdvertiserAssistant`](https://developer.apple.com/documentation/multipeerconnectivity/mcadvertiserassistant) object. The only difference between these two objects is that the latter provides an UI to accept invitations. If you want to create your own UI to let your user manage their invitations, you can use the former.

Other apps can start looking for advertisers using the [`MCNearbyServiceBrowser`](https://developer.apple.com/documentation/multipeerconnectivity/mcnearbyservicebrowser) or [`MCBrowserViewController`](https://developer.apple.com/documentation/multipeerconnectivity/mcbrowserviewcontroller) objects. These two objects will let you see which devices are advertising the service type you want to connect to. Just like advertiser objects, the latter provides you with a standard UI, but you can build your own UI with the former if you want.

Finally, all apps running an instance of the app have a [`MCPeerID`](MCPeerID) associated to them. This ID is unique to each device.

## Playing with the MultipeerConnectivity Framework

With all that theory out of the way, it's time to write a bit of code. We will explore a few more concepts as we do, so you can understand better how to use this framework.

If you want to use the code here, you may want to get two devices. I will provide a sample project at the end that you can install on two devices so you can see them share data with each other.

### Becoming the Advertiser

We will explore how to become an advertiser using `MCNearbyServiceAdvertiser` first, as this gives us more control over the UI and experience when establishing a session.

Establishing an `MCSession` is a two-step progress. The first step is the **discovery step**. In the discovery step, a device can start looking for devices to connect to (advertisers who have a `MCNearbyNearbyAdvertiser` or `MCAdvertiserAssistant` currently advertising) using the `MCNearbyServiceBrowser` object.

Advertisers can start a session with code like this:

```swift
    var advertiser: MCNearbyServiceAdvertiser?
    let serviceType = "MPCTutorial"
    var myId = MCPeerID(displayName: UIDevice.current.name)

//...

func becomeAdvertiser() {
    let discoveryInfo = [
        "Device Type": UIDevice.current.model,
        "OS": UIDevice.current.systemName,
        "OS Version": UIDevice.current.systemVersion
    ]
    
    advertiser = MCNearbyServiceAdvertiser(peer: advertiserId, discoveryInfo: discoveryInfo, serviceType: serviceType)
    
    advertiser?.delegate = self
    
    advertiser?.startAdvertisingPeer()
}

```

There is a bit going on here. First, we create a dictionary called `discoveryInfo`. During the discovery step, before the devices have had the opportunity to establish a session, they can broadcast limited activity about themselves using this dictionary. In our case we are offering the device name, OS, and OS Version to be seen by other devices who want to connect to us. In certain scenarios this can help provide more information to the devices to ensure they connect to the right one.

When we create our `MCNearbyServiceAdvertiser` object, we need to pass in our `MCPeerID`. We create our peer ID also using the device name. The `discoveryInfo` is the same dictionary we defined earlier. Finally, the `serviceType` can be any string you want, as long as it is a maximum of 15 characters long, ASCII characters only, and/or hyphen.

You should only use the `init(displayName)` initializer when creating a peer locally. You can persist `MCPeerdID` long term to be used later on.

We assign the delegate to self. This is a `MCNearbyServiceAdvertiserDelegate` object that will receive events related to device discovery with devices coming to us. We will implement its only method in a bit.

We call `startAdvertisingPeer()` and we are ready to be seen by other devices. There is a matching `stopAdvertisingPeer()` we can use when we no longer want to be discoverable too.

### Searching for devices to connect to

As part of the **discovery step**, other devices start looking for advertisers. This can be done as easily as:

```swift
var myId = MCPeerID(displayName: UIDevice.current.name)
var browser: MCNearbyServiceBrowser?
var connectedPeer: MCPeerID?

//...

func searchForDevices() {
    browser = MCNearbyServiceBrowser(peer: inviteeId, serviceType: serviceType)
    
    browser?.delegate = self
    
    browser?.startBrowsingForPeers()
}
```

We do not need to provide much info to browsers, as most of the info comes from advertisers. Set the delegate as it will receive all the events related to peer discovery. You can `startBrowsingForPeers()` and `stopBrowsingForPeers()` as you see fit.

Once you start browsing for peers, the browser will call two delegate methods:

```swift
func browser(_ browser: MCNearbyServiceBrowser, foundPeer peerID: MCPeerID, withDiscoveryInfo info: [String : String]?) {
    print("We found a peer!")
    print("ID: \(peerID.displayName)")
    print("Device Type: \(info?["Device Type"] ?? "")")
    print("Version: \(info?["OS Version"] ?? "")")
}

func browser(_ browser: MCNearbyServiceBrowser, lostPeer peerID: MCPeerID) {
    print("We lost peer \(peerID.displayName)")
}
```

So, one by one, you will receive information about peers that show up and peers that disappear, easy as that.

### Inviting devices to connect to us

We are following the manual approach in this article, so we will do the manual connecting process.

Once we find a peer we want to connect to (with `browser(foundPeer:peerID:withDiscoveryInfo`), we need to create a `MCSession`. We use this object to connect to other peers.

```swift
func invitePeerToConnect(peerID: MCPeerID) {
    session = MCSession(peer: myId)
    session?.delegate = self
    self.connectedPeer = peerID
    browser?.invitePeer(peerID, to: session!, withContext: nil, timeout: 30)
}
```

You need to set the session delegate to receive events regarding the session, including when the session state changes and when you receive any data.

A bit of discussion on `MCSession` is in order. `MCSession` has more than one initializer. The second initializer can be used to create secure and encrypted communication channels between both devices. We will not discuss "secure" `MCSession`s in this article, but be aware of the `init(peer:securityIdentity:encryptionPreference` initializer, as there may need a case in which you need to verify a peer and/or you'll have the need to share encrypted information. Encryption handling is very transparent. MCEncryptionPreference is just an enum, and you can use encryption without verifying the peer. in iOS 9 and above, it will require encryption by default.

the `context` parameter is an optional data that you can use to pass anything to provide even more context. **Do not send any sensitive data with this**. The connection has not been established yet, so if you are using encryption, this particular piece of data will not be encrypted.

When you invite an advertiser to connect, the `advertiser` delegate will call the `advertiser(didReceiveInvitationFromPeer peerID:context:invitationHandler` delegate method.

In this example, we will immediately accept the invitation to connect:

```swift
func advertiser(_ advertiser: MCNearbyServiceAdvertiser, didReceiveInvitationFromPeer peerID: MCPeerID, withContext context: Data?, invitationHandler: @escaping (Bool, MCSession?) -> Void) {
    print("Invitation to connect from \(peerID.displayName)")
    print("Accepting invite")
    session = MCSession(peer: advertiserId)
    invitationHandler(true, session)
}
```

### Sharing Data - The Session Phase.

After the advertiser has accepted the invite, the session phase will start. When the connection state has changed, the `session(peer:didChange)` delegate method of `MCSession` gets called. When the state is `.connected`, we are ready to send data.

```swift
func sendImage(toPeer peer: MCPeerID) {
    let bundledImage = Bundle.main.url(forResource: "cucoo", withExtension: "png")!
    let imageData = try! Data(contentsOf: bundledImage)
    DispatchQueue.main.asyncAfter(deadline: .now() + .seconds(5)) {
        try! self.session?.send(imageData, toPeers: [peer], with: .reliable)
    }
}
```

On the receiving device, the `session(_:didReceive:fromPeer:)` from peer will get called, and you can process the image then.

```swift
func session(_ session: MCSession, didReceive data: Data, fromPeer peerID: MCPeerID) {
    print("Did receive data")
    if let imageData = UIImage(data: data) {
        DispatchQueue.main.async {
            self.imageView.image = imageData
        }
    }
}
```

The session can receive other kinds of information as well, and it even supports streaming!

```swift
func session(_ session: MCSession, didReceive stream: InputStream, withName streamName: String, fromPeer peerID: MCPeerID) {
}

func session(_ session: MCSession, didStartReceivingResourceWithName resourceName: String, fromPeer peerID: MCPeerID, with progress: Progress) {
    
}

func session(_ session: MCSession, didFinishReceivingResourceWithName resourceName: String, fromPeer peerID: MCPeerID, at localURL: URL?, withError error: Error?) {
    
}
```

# Sample Project

I have created a sample project of everything here. You can download it from [here](/archives/MultipeerApp.zip). You need to install it on two devices to see how it works. The UI simply contains three buttons: One to become an advertiser, one to search for devices, and another to send a default image. When you search for a peer, it will automatically send an invite to the first device it sees. After you tap "Search for Devices", wait a few seconds and tap "send image" on either device. A glorious image of a cucoo will show up on the destination device.

# Conclusion

MultipeerConnectivity provides an easy interface to share data between devices. It will automatically choose the right medium to send data. There' a few things to keep in mind:

* It currently supports 8 peers connected at the same time.
* When the state changes to `.connected`, the connection only lasts a bit when idle. You should try to send data as soon as the connection is established.
* We can get it to work with Bonjour and other APIs if we do manual peer management. That is out of the scope of this article.

# Conclusion

Reflection is a very interesting feature that allows to create some sort of *meta-programming* in Swift. While not applicable to many use cases, it's important to be aware of its existence.

