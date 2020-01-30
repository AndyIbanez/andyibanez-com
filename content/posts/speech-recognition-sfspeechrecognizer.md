---
title: "Recognizing Speech Locally on an iOS Device Using the Speech Framework"
date: 2020-01-29T07:00:00-04:00
originalDate: 2020-01-23T14:52:36-04:00
publishDate: 2020-01-29T07:00:00-04:00
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
 - watchOS
 - iOS13
 - wwdc2019
 - speech
 - speech recognition
categories:
 - development
description: "How to use the Speech framework to detect speech on iOS."
keywords:
 - swift
 - ios
 - ipados
 - watchos
 - iOS13
 - wwdc2019
  - speech
 - speech recognition
---

As iOS becomes more advanced, features that we thought belonged to the long future start becoming more common place in today's software. One such feature is speech recognition, which allows a device to take verbal input from a user, transcribe it into text, and do something with it.

In iOS, we can do this using a framework called `Speech`, and an object called `SFSpeechRecognizer`. With this class, you can perform all kinds of speech recognition tasks.

`SFSpeechRecognizer` supports many languages (far from them all, though), and you can specify which one to use. It also supports different audio inputs of audio to recognize the speech from. So you can choose to recognize speech from a file, or from the device's microphone.

# Implementing SFSpeechRecognizer

## Initial Setup

The `Speech` framework is one of those tools that require you set a string letting your user know what you are going to recognize the speech for. So in your app's `Info.plist`, add the key `NSSpeechRecognitionUsageDescription` of type `string`, and add a short text describing what you are going to use it for.

Then we need to actually ask for permission. The following method will request for permission and return the status of the operation:

```swift
SFSpeechRecognizer.requestAuthorization { (status) in
	switch status {
	case .notDetermined: print("Not determined")
  case .restricted: print("Restricted")
  case .denied: print("Denied")
  case .authorized: print("We can recognize speech now.")
  @unknown default: print("Unknown case")
  }
}
```

You should also check if the speech recognizer is available before you try to use it. For that, instances of `SFSpeechRecognizer` have a property called `isAvailable` you can use to quickly check for availability.

```swift
    if let speechRecognizer = SFSpeechRecognizer() {
      if speechRecognizer.isAvailable {
        // Use the speech recognizer
      }
    }
```

There is also a `supportsOnDeviceRecognition` property. When this is true, the framework will perform on-device speech recognition. When it isn't, it will use the network and send the input to Apple's servers. Make sure you check if this variable if network usage matters.

One little annoyance is that, to instantiate this class, you need to check for optionals. `SFSpeechRecognizer` has two initializers you can use. The default one attempts to construct an object with the device's default language, and if that fails it will try to use the language used for keyboard string recognition. When both conditions fail, it can return `nil`.

The other initializer takes a `Locale` object. This method can also return `nil` when you pass a locale that isn't available. To see all the available locales, you can call the `supportedLocales()` method, which will return a set of locales you can instantiate a `SFSpeechRecognizer` with.

With that out of the way, we can start using `SFSpeechRecognizer` now.

## Recognizing Speech

### Speech Recognizing Tasks

After you have created a `SFSpeechRecognizer` object, you instruct it to execute tasks, which are subclasses of `SFSpeechRecognitionRequest`. At the time of this writing, there's two possible tasks: [`SFSpeechURLRecognitionRequest`](https://developer.apple.com/documentation/speech/sfspeechurlrecognitionrequest), to recognize speech in local files, and [`SFSpeechAudioBufferRecognitionRequest`](https://developer.apple.com/documentation/speech/sfspeechaudiobufferrecognitionrequest), which can take a constant input of audio to recognize speech.

The `SFSpeechRecognitionRequest` superclass offers a few interesting properties you can use to configure your tasks. You can force the request to use on-device speech recognition by setting `requiresOnDeviceRecognition` to `true`; you can force it to report partial results with `shouldReportPartialResults`, and you can provide an array of phrases that should be recognize even when they don't exist in the system's vocabulary with `contextualStrings`. This last one is interesting because you can even make it recognize made-up words. The documentation recommends you keep these to 100 or less.

#### Recognizing Speech in Audio Files with SFSpeechURLRecognitionRequest

Remember to check if the speech recognizer is actually available.

```swift
if recognizer!.isAvailable {
  // Use the speech recognizer
}
```

Once you know it is, you can start using it.

When you call `recognitionTask`, you specify a recognition handler. This is an asynchronous operation that will call you back when it has recognized more speech. If you don't want to use completion handlers, you can use a [SFSpeechRecognitionTaskDelegate](https://developer.apple.com/documentation/speech/sfspeechrecognitiontaskdelegate) instead.

Using this request is very straightforward. All you need to do is to specify the URL to the file in the constructor. You can then configure the `recognitionTask` with the parameters you need. In our case, we will force it to use on-device speech recognition.

Then we call the `recognitionTask` on the your recognizer, and specify a callback for success or error. If you get a correct result you can grab the transcribed results.

The below example will recognize text in a generic audio file, and print its contents:

```swift
  private var recognitionTask: SFSpeechRecognitionTask?
  
  func recognizeFromFile() {
    let fileUrl = // ... URL To file
    let request = SFSpeechURLRecognitionRequest(url: fileUrl)
    speechRecognizer?.supportsOnDeviceRecognition = true
    speechRecognizer?.recognitionTask(
      with: request,
      resultHandler: { (result, error) in
        if let error = error {
          // handle error
        } else if let result = result {
          print(result.bestTranscription.formattedString)
        }
    })
  }

```

#### Recognizing Speech in Audio Streams with SFSpeechAudioBufferRecognitionRequest

If you need to capture audio from a real time source, such as the user's microphone, you can use this request. It works very similar to `SFSpeechURLRecognitionRequest`, but you need to explicitly end the recognition by calling the `endAudio()` method.

The following example will recognize speech provided from the device's microphone. Brace yourself, as this code is much longer than the previous one.

```swift
let speechRecognizer = SFSpeechRecognizer()!
var recognitionRequest: SFSpeechAudioBufferRecognitionRequest?
var recognitionTask: SFSpeechRecognitionTask?
let audioEngine = AVAudioEngine()
func startRecording() throws {
  
  // Cancel the previous recognition task.
  recognitionTask?.cancel()
  recognitionTask = nil
  
  // Audio session, to get information from the microphone.
  let audioSession = AVAudioSession.sharedInstance()
  try audioSession.setCategory(.record, mode: .measurement, options: .duckOthers)
  try audioSession.setActive(true, options: .notifyOthersOnDeactivation)
  let inputNode = audioEngine.inputNode
  
  // The AudioBuffer
  recognitionRequest = SFSpeechAudioBufferRecognitionRequest()
  recognitionRequest!.shouldReportPartialResults = true
  
  // Force speech recognition to be on-device
  if #available(iOS 13, *) {
    recognitionRequest!.requiresOnDeviceRecognition = true
  }
  
  // Actually create the recognition task. We need to keep a pointer to it so we can stop it.
  recognitionTask = speechRecognizer.recognitionTask(with: recognitionRequest!) { result, error in
    var isFinal = false
    
    if let result = result {
      isFinal = result.isFinal
      print("Text \(result.bestTranscription.formattedString)")
    }
    
    if error != nil || isFinal {
      // Stop recognizing speech if there is a problem.
      audioEngine.stop()
      inputNode.removeTap(onBus: 0)
      
      recognitionRequest = nil
      recognitionTask = nil
    }
  }
  
  // Configure the microphone.
  let recordingFormat = inputNode.outputFormat(forBus: 0)
    // The buffer size tells us how much data should the microphone record before dumping it into the recognition request.
  inputNode.installTap(onBus: 0, bufferSize: 1024, format: recordingFormat) { (buffer: AVAudioPCMBuffer, when: AVAudioTime) in
    recognitionRequest?.append(buffer)
  }
  
  audioEngine.prepare()
  try audioEngine.start()
}

```

*(Code provided and adapted from [Apple](https://stackoverflow.com/questions/11525942/play-audio-ios-objective-c))*

That code is a bit of a mouthful, because it uses Apple's AVFoundation framework, which is used to get audio and visual input from a device.

AVFoundation is out of the scope of this article, so we will briefly describe the relevant parts.

When we configure the recognizer, we will use the locale on the user's device. In my case my phone is set to English, so it works on my device. You may need to specify the locale, in case the recognizer does not work with the one specified on your device.

The completion handler in the recognition handler will be report back results received from the microphone. Because we set `shouldReportPartialResults` to `true`, as you speak, the device will print results on the console before you are done talking. On iOS 13 we will force the device to use on-device speech recognition setting `requiresOnDeviceRecognition` to true. The speech recognizer will also know when it has finished recognizing, so you can stop the session when you receive that.

For the microphone's configuration, we will dump the contents of the buffer every 1024 bytes. Strictly speaking, The Speech Recognizer cannot detect speech in real time. The microphone will dump its recent speech into it when the buffer fills to the size you specified. When we dump the contents of the session, the recognizer will recognize the last text dumped into it (by calling `append`). If you make the buffer bigger, it may make this process slower, but it will recognize more text which *may* make it more accurate. But if you make it smaller the recognizer may not be able to keep up. I found 1024 bytes is a good size for this.

## Getting More Data Out of a Recognition Task

You can get more data out of the result of a recognition task. This data includes the speaking rate (the number of words spoken per minute) (`SFSpeechRecognitionResult.speakingRate`); Additionally, the result contains a property called `segments`, which is an array of `SFTranscriptionSegment`). Segments contain data about parts of the spoken text, such as the `confidence` level, which gives us how much a word is likely to match the spoken word. The `timestamp` and `duration` properties tell you the position of the segment in the audio stream, and a `voiceAnalytics` (`SFVoiceAnalytics`) from which you can get the `pitch`, `jitter`, and `shimmer`. You can build very interesting apps with these properties. 

## Additional Setup Options

If you liked what you say, take a look at the [documentation](https://developer.apple.com/documentation/speech/sfspeechrecognizer). There's some more configuration options you can use, including the queue where recognition handlers should be executed on. You can also provide a `defaultTaskHint`, which can help the recognizer be slightly more accurate.

## Gotchas

Keep in mind that not all setups will support on-device speech recognition. The first requirement is to support iOS 13. I haven't found any cases in which the device supports iOS 13 and not on-device speech recognition, but you should always add the proper checks for this in your code.

Also remember that not all languages are supported just yet. You should always check if your user's locale matches one of the values provided by `SFSpeechRecognizer.supportedLocales()`.

You need to keep your audio duration sessions to one minute at most. Speech recognizing can take a lot of battery, and in the case of not having on-device speech recognition, high network usage. The framework will stop recognizing past the one-minute mark.

# Conclusion

Despite a few limitations, on-device speech recognition is finally a thing. While this feature is available on iOS 13, previous iOS devices can use speech recognizing, although it will require an internet connection to work. The API is very easy to use, and you have the option to provide different input sources.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.