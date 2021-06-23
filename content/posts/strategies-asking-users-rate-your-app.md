---
title: "Strategies For Asking Users to Rate Your App"
date: 2021-06-02T07:00:00-04:00
originalDate: 2021-05-31T12:56:24-04:00
publishDate: 2021-06-02T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - apple
 - programming
categories:
 - development
keywords:
 - swift
 - apple
 - programming
---

Having our apps have good reviews is generally a good thing. After all, many users look into how many stars an app has before deciding on downloading it. Apps with a general poor rating may not get many downloads (unless they are "essential" apps of any kind, such as companion app to another service).

We all as developers have experienced that users are quick to give a one-star review when something doesn't work right, but they are never inclined to rate 5 stars when they are satisfied with an app.

Luckily  for us, Apple has given us the `SKStoreReviewController.requestReview()` API. Introduced in iOS 13, we can call this method (rather, its non-deprecated sibling, `SKStoreReviewController.requestReview(scene:)`) to prompt users to rate our app.

[Request Rating Review](/img/review_request_prompt.png)

Of course the ability to ask for ratings does come with some restrictions. First, users can turn off this prompt completely at the system level, and second, while you can call this method as many times as you like, it will only be displayed to your user at most 3 times per *year*. So it's not a good idea to go around willy nilly calling it everywhere. Instead, you need to think of ways to call this method without being annoying and without wasting the alert's presentation.

In this article we will talk about the do's, don'ts, and other strategies you can use to decide when you should attempt to show a review prompt to your users. We are not going to talk about the technicalities of the API as it is almost literally a one-line call - I say almost, because on the non-deprecated version of the method you do need to get a scene to call the alert on. So we before we dive in, allow me to give you this barebones piece of code that you can use to quickly get a scene and call the review prompt on it.

```swift
func showReviewPrompt() {
    if let scene = getScene() {
        SKStoreReviewController.requestReview(in: scene)
    }
}

func getScene() -> UIWindowScene? {
    if let iPadScene = getIPadScene() {
        return iPadScene
    } else {
        return getIPhoneScene()
    }
}

func getIPadScene() -> UIWindowScene? {
    UIApplication.shared.connectedScenes.first(where: { $0.activationState == .foregroundActive }) as? UIWindowScene
}

func getIPhoneScene() -> UIWindowScene? {
    UIApplication.shared.connectedScenes.first as? UIWindowScene
}
```

# What you should NOT do

Before we dive into a couple of strategies I have used in the past with good success, let's talk about what you shouldn't do. I want to kick off this article with the don'ts because it's a mistake I see a lot of developers make.

## Attempting to show the prompt on launch

Many developers decide to call the review prompt when an app launches and call it a day. This is not really a good strategy for two reasons:

1. Users may not mind giving apps a rating, but they may have a sense of urgency using your app and they are not interested in seeing the prompt first thing.
2. You will likely show the prompt before the user has had a chance to use your app.

In the best scenario, users will dismiss the prompt and move on. You will have wasted one of your three displays in a year, but no harm will be done. Worst case scenario, users will give you a 1-star rating if they see your prompt when launching your app.


## Showing the prompt too soon, before users have a chance to use your app.

Probably the downside of the requestReview API is that you need to manually code all the checks in order to control if the system will attempt to show the alert or not. Many times I download an app and it either asks me for a review as soon as I launch it the second time or after a few minutes of use. I tend to dismiss all the prompts that occur too soon because I simply don't know what rating an app deserves. They will show the prompt three times and I will end up dismissing them all. The app then loses my rating.

In my case, I dismiss the prompt and move on. Beware of users who will happily give you a one-star review for being persistent.

At the very least, you should prevent the alert from being shown on the same day the user installs the app. I'd program a timer to start asking for ratings after at least one week has passed.

## Showing the prompt upon any generic action.

Ideally, you should look for an action users perform often enough that has some value to them, and attempt to prompt a rating after the user has done it. Launching the app and opening a screen are just too random to ask for a rating. In generally if users feel satisfied and they feel a specific action has given them value, they will be more likely to rate your app with a positive score.

# Strategies for getting the best out of review prompts

I can't stress this enough, but you have three opportunities to ask your users for a rating in a 365-day cycle. You should avoid wasting these opportunities at all costs. Actually having an strategy that governs your review prompt display can yield many positive results.

## Look for specific satisfying actions to make the call on

If you do not want to spend much time thinking on a strategy for your review prompt, spend a few hours thinking on the key areas or your app that bring value to your users, and attach the call to those actions. Don't blindly put your calls when launching your app and call it a day.

## Piggyback on existing score prompts to call the review prompt

At my day job, I maintain the iOS app for a bank, and one of the things we do, is to ask users what their experience was like after doing a transfer. We ask them every 14 days after the last transfer. This prompt is not linked to the App Store prompt. Instead, it's linked to our internal analytics database.

If you have something like that, feel free to piggy back on it and show a review prompt if the user has given a positive score to your previous prompt. In general, users who had a positive experience and are likely to say they had a good experience with something, they are likely to do it again even if they were just asked a second ago. Very few users may still give you 1-star reviews, but the vast majority of them who are satisfied will not.

## Implement a weight-based system to govern the display of your prompt

If you have an app that is designed to be used for long periods of time, or if you have an app that may have users performing multiple actions in a short session, you can implement a weigh based system.

The idea behind this system is that you assign a *weight* - a number - to different actions a user may perform. For example, suppose you wanted to implement this system on the Twitter app:

- Opening a Tweet has a weight of 1 point.
- Loving a tweet has a weight of 3 points.
- Creating a tweet has a weight of 5 points.

You then decide on a threshold, and when this threshold is reached, you show the prompt.

In this way, suppose you have a threshold of 40. Your users can open the app and perform the following actions before the prompt is shown:

- User launches the app.
- User likes 3 recent tweets of their favorite celebrity. (+9 points)
- User opens two celebrity tweets to check the replies. (+11)
- User decides to reply with one tweet to their favorite celebrity, and another reply to a random commenter in the twitter thread. (+10)
- User closes the app for the day.
- The next day, the user makes a random tweet (+5)
- The user likes two tweets from another celebrity (+6)

After all these actions take place, weight will reach 41. The prompt is displayed, and since it's established your user likes the app, they are likely to give it a good rating.

You can play around and mix and match any other constraints to this prompt. For example, you may want to avoid showing the prompt until at least one week has passed since your user first used your app.

# Conclusion

Apple gives us the tools to aid us improve the rating of our app. This can have long-term implications, make your app more popular, and more. But you have to use this tool very effectively, because it's very limited, and if you are not careful, it may actually backfire on you.

