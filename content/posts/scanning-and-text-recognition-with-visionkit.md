---
title: "Document Scanning and Text Recognition With Vision and VisionKit on iOS"
date: 2020-06-10T07:00:00-04:00
originalDate: 2020-06-07T12:33:42-04:00
publishDate: 2020-06-10T07:00:00-04:00
draft: true
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - visionkit
 - ocr
 - scanning
 - vision framework
categories:
 - development
keywords:
 - swift
 - programming
 - apple
 - ios
 - ipados
 - visionkit
 - ocr
 - scanning
 - vision framework
description: "Learn how to scan documents and detect in iOS with the Vision Framework."
---

It is amazing what we can do with smartphones these days. Document scanning and text recognition are nothing new. But being able to have such a functionality in our pockets is pretty neat. These days we can create apps that have such features very quickly thanks to the push Apple has been doing to promote Machine Learning and Artifical Intelligence on their devices.

Starting on iOS 11, we can natively scan documents with a system framework called VisionKit, and we can perform operations on images using a framework called Vision. It wasn't until iOS 13 that we finally had the ability to recognize text on images ourselves using the Vision framework, without leveraging third party libraries. In this article we will explore how we can use the VisionKit framework to scan documents and the Vision framework to detect text as two separate tasks, so you can see how easy these two tasks are and you can learn to put them together.

# Introducing the VisionKit Framework

Apple introduced the Vision framework on iOS 11 and it offers very quick APIs to do document scanning. The framework comes with a view controller that you can use to do the scanning, so you don't have to worry about wiring your own UI for this. The framework comes with a delegate so you can be notified as scanning events take place.

## Scanning Documents with VNDocumentCameraViewController

`VNDocumentCameraViewController` is the object we use to perform scanning. Setting it up is pretty straightforward. This object comes from the `VisionKit` framework, so make sure you import that. You will also need to comform to `VNDocumentCameraViewControllerDelegate` in order to receive events.

```swift
import VisionKit

class ViewController: UIViewController, VNDocumentCameraViewControllerDelegate {
	var documentCamera: VNDocumentCameraViewController?

	//...

	func documentCameraViewControllerDidCancel(_ controller: VNDocumentCameraViewController) {
	  documentCamera?.dismiss(animated: true, completion: nil)
	}
	  
	func documentCameraViewController(_ controller: VNDocumentCameraViewController, didFailWithError error: Error) {
	  print("Document Scanner did fail with Error")
	}
	  
	func documentCameraViewController(_ controller: VNDocumentCameraViewController, didFinishWith scan: VNDocumentCameraScan) {
	  print("Did Finish With Scan.")
	}

	//...
}
```

When you actually want to show a document scanner, just instantiate a new `VNDocumentCameraViewController`, assign your delegate, and present it:

```swift
func showDocumentScanner() {
  guard VNDocumentCameraViewController.isSupported else { print("Document scanning not supported"); return }
  documentCamera = VNDocumentCameraViewController()
  documentCamera?.delegate = self
  present(documentCamera!, animated: true, completion: nil)
}
```

Don't forget to set `NSCameraUsageDescription` in your plist, otherwise your app will crash when you try to present it.

We will implement the logic to get the images in a sec, but for now, run it and try scanning something with it:

The scanner automatically grabs what it thinks is a document and scans it, and it automatically starts creating new pages based as more content is scanned. Pretty neat!

Now we care about receiving the images, and for that, we will write some code in the `documentCameraViewController(controller:didFinishWith:)` method. This method gives us a `VNDocumentCameraScan` we can use and it has all the information about the scanning process, including the number of pages, and the images themselves. This object has two mere properties: `title`, `pageCount`; and a single method `imageOfPage(at:) -> UIImage`

```swift
func documentCameraViewController(_ controller: VNDocumentCameraViewController, didFinishWith scan: VNDocumentCameraScan) {
  documentCamera?.dismiss(animated: true, completion: nil)
  documentCamera = nil
  print("Finished scanning document \"\(String(describing: title))\"")
  print("Found \(scan.pageCount)")
  let firstImage = scan.imageOfPage(at: 0)
  // .. Do something with the first image
}
```

And that's it! You now know how to quickly scan documents and get results back.

# The Vision Framework

The Vision Framework contains APIs to let us analize images.

# Detecting Text in Images Using the Vision Framework.

## VNRecognizeTextRequest

To use this framework, you create requests and handlers. A request performs the operation you want, and then you hand the request to a handler to execute your request.

We will start by creating our recognition request:

```swift
func detectText(in image: UIImage) {
  guard let image = image.cgImage else {
    print("Invalid image")
    return
  }
  

  let request = VNRecognizeTextRequest { (request, error) in
    if let error = error {
      print("Error detecting text: \(error)")
    } else {
      self.handleDetectionResults(results: request.results)
    }
  }
  
  request.recognitionLanguages = ["en_US"]
  request.recognitionLevel = .accurate
  
  performDetection(request: request, image: image)
}
```

First we need to convert out `UIImage` to `CGImage`, as we will need it later. Then, when we create our `VNRecognizeTextRequest`, the initializer takes a completion handler that gets called when the operation is completed. You can set a couple of properties in the recognition object to help it be more accurate, like the recognition language. You can set the recognition speed which allows you to specify if you want to be accurate or fast.

To actually execute the task, we will implement `performDetection(request:image)` as so:

```swift
func performDetection(request: VNRecognizeTextRequest, image: CGImage) {
  let requests = [request]
  
  let handler = VNImageRequestHandler(cgImage: image, orientation: .up, options: [:])
  
  DispatchQueue.global(qos: .userInitiated).async {
      do {
          try handler.perform(requests)
      } catch let error {
          print("Error: \(error)")
      }
  }
}
```

The `VNImageRequestHandler` will actually execute our `VNRecognizeRequest`. You want to do this in another thread, as it can be a lengthy operation. When we create our object, we need to pass it the image, and the image orientation. It looks like the recognizer is not smart enough to recognize the orientation of images, so you need to tell it what position your text is in, probably providing a way in your UI to do just that.

When our request finishes, we will call `handleDetectionResults(results:)`, which is implemented as:

```swift
func handleDetectionResults(results: [Any]?) {
  guard let results = results, results.count > 0 else {
      print("No text found")
      return
  }

  for result in results {
      if let observation = result as? VNRecognizedTextObservation {
          for text in observation.topCandidates(1) {
              print(text.string)
              print(text.confidence)
              print(observation.boundingBox)
              print("\n")
          }
      }
  }
}
```

Everything this method does is to print the strings and how much confidence it has in them. The confidence is how accurate the recognizer thinks it is.

A candidate is a piece of string of text the recognizer thinks it found. It's probably not practical to get many candidates per iteration, but you should evaluate according to your needs. If the text is clear enough, you can get by with the first or first two candidates. The bounding box is the coordinates in the image where it found the strings.

And that's it! Using these APIs is actually very easy thanks to Apple's push on machine learning. The results are very accurate most of the time, despite it using on-device machine learning instead of a whole cloud like Google backing it up. The API is trustworthy, and very powerful.

# Conclusion

We can create scanning and OCR apps very easily with the use of these APIs on iOS and iPadOS. Apple provides many and high-level APIs for these tasks that are easy to use and are also very fast and accurate.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.
