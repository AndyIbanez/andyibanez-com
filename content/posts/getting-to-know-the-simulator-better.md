---
title: "Getting to Know the Simulator Better"
date: 2021-03-10T07:00:00-04:00
originalDate: 2021-03-06T18:18:20-04:00
publishDate: 2021-03-10T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - apple
 - simulator
 - ios
 - ipados
 - watchos
 - tvos
categories:
 - development
keywords:
 - apple
 - simulator
 - ios
 - ipados
 - watchos
 - tvos
description: "Learn about the features the Simulator offers to make your Apple App development better."
---

Every iOS developer has used the simulator. Alongside Xcode, it's probably one of the most used tool by us all. We use the simulator to test our iOS, iPadOS, and watchOS apps without having to run them in an iOS device.

But other than helping us test our apps, the simulator actually has many nice features that can help make our job a little bit easier. With the use of these features we can avoid using physical devices until it's time to do an actual test run or one.

In this article, we will explore the Simulator features provided to us by Xcode and the `xcrun simctl` tool.

# simctl Basics

Before we get into the good stuff, let's explore some basic functionality offered by `simctl`. You can, after all, use this to write some automation for your work with the simulator and it can save you some trips to Xcode's GUI.

## Listing Simulator Devices

You can list the available simulator runtimes by simply running the `list` subcommand.

```
xcrun simctl list
```

This will print a lot of stuff, but among the data, you will find the name of the emulated device and the OS version is currently emulating.

```
== Devices ==
-- iOS 13.2 --
    iPad Pro (9.7-inch) (5FB75994-26F5-4455-80E7-14D00CBACA81) (Shutdown) 
    iPad (7th generation) (7055FC4A-DC49-433B-8005-13E9AC2CDCF7) (Shutdown) 
    iPad Pro (11-inch) (1st generation) (8D50B34C-FABE-41D0-807E-6AB5CAD73E41) (Shutdown) 
    iPad Pro (12.9-inch) (3rd generation) (FD578FA9-987A-40A9-8317-695E40D2D52C) (Shutdown) 
-- iOS 14.4 --
    iPhone 11 Pro Max (0DD83491-914D-4EB9-9B8A-00A3191199B7) (Shutdown) 
    iPhone SE (2nd generation) (9C8A6939-9AC5-4D7D-AA46-9E8DA7F22725) (Shutdown) 
    iPhone 12 mini (0F59FD9B-DC0C-4812-99EF-E88236667AB9) (Booted) 
    iPhone 12 (31891A98-684F-4C82-9767-EF09FD91F678) (Shutdown) 
```

It even lists device pairs, which are used when you are working with an Apple Watch companion app to your iPhone app:

```
== Device Pairs ==
3A322BC4-879B-49B8-B60D-6C7647B2C567 (active, disconnected)
    Watch: Apple Watch Series 5 - 40mm (000B4763-2096-434E-97E4-74154DD72B4E) (Shutdown)
    Phone: iPhone 12 mini (0F59FD9B-DC0C-4812-99EF-E88236667AB9) (Booted)
AFB5C78F-8C41-4851-98F7-39D5A49F8E5C (active, disconnected)
    Watch: Apple Watch Series 5 - 44mm (EF2AD5C2-16B5-41D9-AE75-3EE657F0630F) (Shutdown)
    Phone: iPhone 12 (31891A98-684F-4C82-9767-EF09FD91F678) (Shutdown)
```

All the devices have a unique (UUID) identifier associated to them. When using `simctl`, you specify the UUID you want the command to act upon.

## Booting Devices

Use `xcrun simctl boot YOUR_UUID_ID` to boot a device.

Here, I'm booting my iPhone 11 Pro Max simulator running iOS 14.4.

```
xcrun simctl boot 0DD83491-914D-4EB9-9B8A-00A3191199B7
```

Note that this command will send the boot up signal to the simulator and return immediately return. In other words, there's no way to "wait" until the device is ready for use and is fully booted up.

Earlier, we said that `simctl` takes an UUID on its commands. After a device is booted, you can pass in the word `booted` instead of the UUID and the simulator will automatically choose the currently booted device to execute the commands on. If you are running multiple simulators and you pass in `booted`, the command line tool will select one of the booted simulators, so if you have multiple simulators open, it's best to keep passing the respective UUIDs.

Once a simulator is booted, their status on `simctl list` will change to `(Booted)` instead of `(Shutdown)`.

```
-- iOS 14.4 --
    iPhone 7 (4C1A1BE4-82D0-428F-9B22-0CC1FC776821) (Shutdown) 
    iPhone 8 (4A3B434B-1EAA-4D30-BD4C-4C03906CA6E6) (Shutdown) 
    iPhone 8 Plus (1A3195D3-2557-4B8C-85C8-C6C49E0350BC) (Shutdown) 
    iPhone 11 (03BBFEB2-7DCB-41D1-A49D-18AB55AB3C0F) (Shutdown) 
    iPhone 11 Pro (726B8086-CBF5-4FE4-B0BB-DFE2100D95F0) (Shutdown) 
    iPhone 11 Pro Max (0DD83491-914D-4EB9-9B8A-00A3191199B7) (Booted) <--------
```

This is just bash, so you can combine other shell commands through piping. For example, we can choose to list only the devices that are currently booted:

```
xcrun simctl list | grep "(Booted)"
```

```
iPhone 11 Pro Max (0DD83491-914D-4EB9-9B8A-00A3191199B7) (Booted) 
iPhone 12 mini (0F59FD9B-DC0C-4812-99EF-E88236667AB9) (Booted) 
```

## simctl with JSON

You can make the `list` and other subcommands print in json by adding the `--json` (`-j` for short) flag.

```
xcrun simctl list --json
```

```
  "runtimes" : [
    {
      "bundlePath" : "\/Library\/Developer\/CoreSimulator\/Profiles\/Runtimes\/iOS 13.2.simruntime",
      "buildversion" : "17B102",
      "runtimeRoot" : "\/Library\/Developer\/CoreSimulator\/Profiles\/Runtimes\/iOS 13.2.simruntime\/Contents\/Resources\/RuntimeRoot",
      "identifier" : "com.apple.CoreSimulator.SimRuntime.iOS-13-2",
      "version" : "13.2.2",
      "isAvailable" : true,
      "name" : "iOS 13.2"
    },
    {
      "bundlePath" : "\/Applications\/Xcode.app\/Contents\/Developer\/Platforms\/iPhoneOS.platform\/Library\/Developer\/CoreSimulator\/Profiles\/Runtimes\/iOS.simruntime",
      "buildversion" : "18D46",
      "runtimeRoot" : "\/Applications\/Xcode.app\/Contents\/Developer\/Platforms\/iPhoneOS.platform\/Library\/Developer\/CoreSimulator\/Profiles\/Runtimes\/iOS.simruntime\/Contents\/Resources\/RuntimeRoot",
      "identifier" : "com.apple.CoreSimulator.SimRuntime.iOS-14-4",
      "version" : "14.4",
      "isAvailable" : true,
      "name" : "iOS 14.4"
    },
```

## Shutting Down And Clearing Simulators

You can close all simulators and even clear their contents and settings with two simple commands:

```
xcrun simctl shutdown SIMULATOR_UUID
xcrun simctl erase SIMULATOR_UUID
```

## Opening URLs in the Simulator

If you have been using Safari to try out your custom URL schemes, you can stop doing that now. There is a command that will prompt the simulator to open a URL and it will route the request to the appropriate app.

```
xcrun simctl openurl (SIMULATOR_UUID|booted) maps://
```

This is not limited to schemes, as it can open normal HTTP links as well in the default browser.

```
xcrun simctl openurl (SIMULATOR_UUID|booted) "https://www.andyibanez.com"
```

## Screenshots and Videos

You can take screenshots from the simulator in two ways:

* Press Cmd + S
* Using the `xcrun simctl io SIMULATOR_UUID|booted screenshot FILE_NAME.png`

You can also record video with the following command:

```
xcrun simctl io booted SIMULATOR_UUID|booted recordVideo FILE_NAME.mp4
```

This will begin recording indefinitely. Press `Ctrl + C` on the console when you are done recording, and the video will be saved on your Mac.

You can pass in some additional flags to `recordVideo`. We can, for example, change the codec altogether and even the device mask.

```
xcrun simctl io booted recordVideo --codec h264 --mask ignored myVid.mp4
```

### Capturing Video on External Displays

The simulator can simulate external displays. If your app supports them, you can choose whether to capture from the main device or an external device.

To quickly test this, we can use the Photos app.[^]

Go to `I/O` > `External Displays` on the Simulator menu to explore the different options you have.

## Adding Media to the Photos.app

There's two ways you can add media to the Photos.app

The first one is the following command:

```
xcrun simctl addmedia SIMULATOR_UUID|booted ~/path/to/file/pic.jpg
```

This is a great way to script and automate the addition of new media.

But if you just need to quickly add media to the simulator, you can now simply drag and drop the media from anywhere in your Mac to the simulator.

To capture from an external display:

```
xcrun simctl io booted recordVideo --display external externalVid.mp4
```

# Privacy

As you know, on iOS we don't have free access to all the areas on the operating system as Apple takes user privacy very seriously. Instead, when an app needs access to data deemed private, the user will see a prompt asking them if it's OK to grant your app that data. This data includes, but is not limited to, contacts, calendar, and health.

We can quickly grant access to this personal data on the simulator to your app by running:

```
xcrun simctl privacy SIMULATOR_UUID|booted grant APP_NAME YOUR_APP_BUNDLE_ID
```

Where:

* APP_NAME is the name of the data with private data you want to access, such as `contacts`, `calendar`, `health`, `wallet`, and more. If you want to grant access to everything, you can simply write `all`.
* YOUR_APP_BUNDLE_ID is the bundle ID of the app that will have access to this data.

Granting and revoking is equally easy. To revoke permissions, replace `grant` with `revoke` in the command above.

On iOS, permissions normally have three states: undefined, granted, and denied. The prompt is shown only when the state is undefined. After a user grants or denies a permission, the prompt will show.

You can reset all the permission settings to the "undefined" state and force the system to show a prompt for your app by running:

```
xcrun simctl privacy SIMULATOR_UUID|booted reset all
```

Just like above, you can replace `all` with specific apps.

# Device Specific Features

## iPadOS

The iPadOS simulator allows you to test the mouse integration with your app. By clicking the "Capture Pointer" button on the simulator, you will be able to simulate using a mouse in your app:

![iPadOS Simulator Bar](/img/ipad_sim_bar.png)

When you want to exit, press the `Esc` key.

A lot of this is actually customizable. If you need to do something with the `Esc` key on your iPadOS app, you can configure a different key to stop capturing your mouse. For this, go to `Simulator` > `Preferences` and choosing another option for `Stop capture shortcut`.

# Newly-added Features on iOS 14.

Xcode 12 and iOS 14 and the rest of the OS family bring a few new features to the simulator, most notably:

## Creating new Simulators without Xcode

You can go to `File` > `New Simulator...` to create a new simulator quickly and painlessly. You will be asked for a name, type, and OS version.

## Push Notifications

New (and very exciting) to Xcode 12, we can send push notifications to the simulator without having to use a server.

To trigger the notifications, you can do it in two ways:

* With the `simctl` `push` command:

```
xcrun simctl push SIMULATOR_ID|booted YOUR_APP_BUNDLE_ID PAYLOAD_JSON_FILE.json
```

You must specify the bundle ID of your app in order for the command to know what App triggered the request.

The `PAYLOAD_JSON_FILE.json` parameter is a file that contains a JSON in the same format as push notifications.

It looks like this:

```
{
    "aps" : {
		    "badge" : 1, 
        "alert" : {
            "title" : "New Message",
            "body" : "Sakura: Hey I just captured a new card!"
        }
    }
}
```

This is a simple payload you can use, but Apple documents all the possible fields [here](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/CreatingtheNotificationPayload.html).

# Modifying the Status Bar.

With the simulator, we can even modify the status bar. This is useful when preparing screenshots for the App Store or other marketing material. You may know Apple always show 9:41 in all its screenshots when the status bar clock is visible - this is because the first iPhone was shown to the world for the first time at this time.

But that's not all, you can also make it show full signal, full battery, full Wi-Fi bars, or customize any of these parameters to your liking.

Simply run the `status_bar` subcommand with any or all of the following flags:

```
xcrun simctl status_bar booted override --time 07:00 --cellularBars 3 --dataNetwork LTE --wifiMode failed
```

If you need to override these settings, simply run:

```
xcrun simctl status_bar booted clear
```

If you run the `status_bar` subcommand without any arguments, you will be able to see all the possible flags you can modify. It's very powerful and flexible, as you can even modify the operatorName:

```
xcrun simctl status_bar
```

At the time of this writing, this printed:

```
Set or clear status bar overrides
Usage: simctl status_bar <device> [list | clear | override <override arguments>]

Supported Operations:
    list
	List existing overrides.

    clear
	Clear all existing status bar overrides.

    override <override arguments>
	Set status bar override values, according to these flags.
	You may specify any combination of these flags (at least one is required):

	--time <string>
	     Set the date or time to a fixed value.
	     If the string is a valid ISO date string it will also set the date on relevant devices.
	--dataNetwork <dataNetworkType>
	     If specified must be one of 'wifi', '3g', '4g', 'lte', 'lte-a', or 'lte+'.
	--wifiMode <mode>
	     If specified must be one of 'searching', 'failed', or 'active'.
	--wifiBars <int>
	     If specified must be 0-3.
	--cellularMode <mode>
	     If specified must be one of 'notSupported', 'searching', 'failed', or 'active'.
	--cellularBars <int>
	     If specified must be 0-4.
	--operatorName <string>
	     Set the cellular operator/carrier name. Use '' for the empty string.
	--batteryState <state>
	     If specified must be one of 'charging', 'charged', or 'discharging'.
	--batteryLevel <int>
	     If specified must be 0-100.
```

# Keychain Management

You can certificates to the device's keychain with the `keychain` command:

```
xcrun simctl keychain booted add-root-cert myCert.cer
```

You can also simply drag and drop your certs.

Keep in mind that you will have to trust the certificate manually, as neither of these options will do that for you.