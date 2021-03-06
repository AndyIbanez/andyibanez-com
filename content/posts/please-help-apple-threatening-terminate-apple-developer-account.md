---
title: "Please Help, Apple Is Threatening To Terminate My Apple Developer Account With No Clear Reason."
date: 2020-06-17T11:22:35-04:00
draft: false
categories:
 - development
---

**UPDATE**

June 18, 2020 at 10:24 PM EST: I have made contact with a human at Apple after they reached to me via Resolution Center. We are working on solving the issues, and while I have not been told "yes, your account will not be terminated", I believe in good faith we will work something out. Yes, the issue was my fault after all, but they are being very understanding and are letting me redeem myself by uploading a new binary. A proper update on this issue will come at a later date.

**SECOND AND FINAL UPDATE**

June 19, 11:47 PM EST:

After talking to Bill from Apple, I managed to avoid getting terminated.

The issue was indeed my fault. Turns out that at some point after the first public TestFlight build, I built some internal features to help me test the app easier. These features were not meant to reach any public builds, but they slipped (guess who didn't know about `#available` a few years ago). One of these internal test tools was the ability to import data so I could test known servers that worked with the app. These testing features were found by the App Review team post-approval (many years after approval, in fact), and they decided to remove the app and flag me for termination because it looked like I was shadily deploying un-reviewed features to my users to bypass the App Store. Couple that with the fact the App had been on TestFlight for three years, and it really looked like I was bypassing the App Store breaking the rules.

I was reached on the Resolution Center of my app after this article blew up and I was given a phone number I could call. I do not know if my article triggered a human to contact me, or if they were going to do it eventually, but I was told they read this. On the first call, I was explained what the problem was. After a few calls I was told that I was being given a second chance by the App Review team, and I have been working to fix the issue since.

It was hard to reach a human at Apple when my account was flagged, but after I finally managed to, everything progressed smoothly.

There are a few lessons to learn from this:

* If you build internal functionality on your app, make sure it is completely removed from public builds.
* Apple can and will re-review apps years after they have been approved to check for compliance. This means both App Store and TestFlight apps. This is a good thing for users, as it helps fight really shady apps that start doing something else, probably with a server flag. We have known that Apple silently reviews App Store apps again years or months after they are approved - now we know this applies to TestFlight builds as well.
* Reaching humans is hard, but extremely rewarding when you manage to do it. To be clear, you can reach a human most of the time when dealing with App Review. They have forms and easy methods to do just that. But when your issue leaves the realm of App Review (in my case, I'm guessing my issue was handled by some sort of Fraud department), it's *hard*.
* This is still speculation, but my guess that the app was on TestFlight for too long wasn't an issue on its own. It's the fact that a internal features were deployed AND it looked like I had been deploying these features secretely for years that got me flagged. I lean towards saying that having an app on TestFlight for years is not an issue on its own, but I will probably refrain from doing that for now on. At the very least, I will remove builds and force them to go through manual review every couple of months. Especially after WWDC, because I will be busy doing changes that work well for iOS 14 and I'm sure the app will be mutating a lot. :-)

In this article, I said that I was ready to accept any punishment done to me if it indeed turned out it was my fault. Turns out I was entirely to blame, but Apple (specifically Bill), managed to prevent my termination from the program. I cannot thank Bill enough for his kindness and understanding, despite the fact that the issue was, in fact, quite big. It's always great when you can reach a human, and it shouldn't be complicated to do when you really need to reach out.

**ORIGINAL POST**

<hr>

Hello guys.

I have been holding off to write this in case I was able to solve this problem on my own (which I tried to do very patiently), but the fact that I cannot talk to a human at Apple has prompted me to write this. Today I received another generic response from them (probably from a bot), so I'm prompted to write this to try to reach out to someone who can help.

Yesterday at around 1PM EST, Apple opened and immediately closed an issue on an app I have on TestFlight. This is important, the app is on TestFlight, and not even the App Store.

I will tell my version of the story, and I sincerely wish Apple would do the same, but they are not providing me with any information that can help me understand what I did wrong.

The app in question is [Mignori](https://www.mignori.com). Mignori is a client for Image Board websites (we informally call "Boorus"). These image boards are popular in the anime community, although there can be image boards for anything you can think of. This app let you browse those websites implementing their APIs so you have access to all their features.

The app does not ship with any servers by default. The user has to add them all manually. Just like a web browser.

While the first version of this app was on the App Store from 2014 to 2016, I removed it in that last year because the app stopped working properly with most Boorus, and it was broken for most users. Since then, I started developing a new version called "Mignori 3", which has *never* hit the App Store. It has been on TestFlight for a while.

Apple cites "Fraud" as the termination reason for my app and therefore my account, and I'm trying to wrap my head around what they think is fraudulent in my app. Especifically, they cite a paragraph of the developer program.

> "You will not, directly or indirectly, commit any act intended to interfere with the Apple Software or Services, the intent of this Agreement, or Apple’s business practices including, but not limited to, taking actions that may hinder the performance or intended use of the App Store, Custom App Distribution, or the Program (e.g., submitting fraudulent reviews of Your own Application or any third party application, choosing a name for Your Application that is substantially similar to the name of a third party application in order to create consumer confusion, or squatting on application names to prevent legitimate third party use)."

My app is image browser app. The business model is clear from the get go. The app was intended to be free, and it was supposed to offer a Pro Version to unlock all the versions, as well as Theme Packs to let users customize the App and me getting a little bit of revenue without having to implement subscriptions. All the network requests the app ever made where the HTTP requests to the image board browsers. Because this app is essentially a web browser.

I am not hindering or interacting with Apple's servers in any way beyond implementing the required APIs for implementing my IAPs. Some of my IAPs use Apple's Hosted Content API, but they aren't that big. They are a few kilobytes in size and they are text files that contain information about the themes. In other words, they are theme files.

Apple cites "being involved with fraudulent reviews" to remove this app. But there's two problems with this. First, the app is on TESTFLIGHT. I cannot receive ratings even if I want to. Second, I do not know of any app that does what mine is supposed to do, and even if I knew one, I am not a lame human being who would try to ruin their reputation with fake negative reviews, especially considering mine is not on the App Store yet and has been under development for four years.

The app name is "Mignori" is pretty unique as far as I know. It is not common enough to have a .com domain, which is why I was able to register it. If I see a problem here, is that "Mignori 3" is a different app than "Mignori", both having two profiles on my developer account and two different bundle IDs. I did this, because when I killed the first version of Mignori, I was thinking some users still had some use for it, and the new version was written from scratch and it is not compatible with the old format at all. Mignori 3 was published on TestFlight in 2017 and Apple approved it. While I haven't been able to reach a production state yet, I have been working with my testers to patch all the bugs, although slowly, because this app is a hobby, and real life happened.

I am not "squatting" the name Mignori because it is not a common name. It is a name I genuinely use. I own the .com domain name for it. And even IF Apple wanted to free this name, I'm sure there are thousands of ways they could do so, without having to threaten to lock down my developer account.

I know other reasons are misleading metadata (which I do not do), "bait-and-switch" schemes (which I would not do, and I CANNOT do because the app is not on the App Store) can also cause this. But I have evaluated the rules over and over again and looked at my TestFlight app and I cannot find a single reason it is considered a "fraud".

The big problem is that when Apple tells you that the reasons "are not limited" the ones they listed, it could be anything. They could call my face ugly and remove me from the Developer Program under that reason and tell me it was because of "fraud".

My intention is not to "Run to the press", like they call it, but when you are stuck in a loop of attempted appeals, you cannot get ahold of a human, and all you get are automatic bot responses, what are your options? I am not a popular developer in the iOS community, so I am not sure how far this post will reach (hopefully far enough, please).

The saddest truth of the matter is that I am not the only guy who has had to dealt with this. Even [Gui Rambo](https://rambo.codes/posts/2019-11-05-apple-has-locked-me-out-of-my-developer-account) has had a problem similar to this, and we all know that Rambo has no reason to do anything illegal with the Apple Developer Program. If you search around, you will find [lots](https://forums.developer.apple.com/thread/77865) [of](https://forums.developer.apple.com/thread/126214) [people](https://forums.developer.apple.com/thread/112036) [with](https://apple.stackexchange.com/questions/360547/any-chance-to-survive-after-apple-developer-program-membership-will-be-terminat) the same problem. I give these people the benefit of doubt that, like me, they did not break the Apple Developer Program rules in a way that would prompt termination of their accounts. None of them could ever reach a human at Apple. The only people I know managed to reach Apple where the really popular developers who were in the eye of the storm, and Apple ended up reaching out to them.

Right now, this is the worst thing that happened to me, and right before WWDC as well. WWDC is my favorite event of the *year*. I look forward to it every single year and even ask for days off at work to watch the Keynote and Platform State of the Union at home. This uncalled for event has completely killed my hype for WWDC, but I will keep my hopes up that this can be rectified before WWDC actually arrives.

# What I Think I Did Wrong

The only thing I can think of I did wrong is I had the app on TestFlight for far too long with a public invitation link. Ever since the app went live on TestFlight, I have had a public link on the Mignori website where people could register to use the beta of the app. Back in 2017, the development of the app was very active and moving smoothly, but in early 2018 real life happened, and I couldn't work on the app as actively as I would have liked. The app has been sitting on TestFlight for 3 years now, it has around 200 beta users, and if anything, the only reason I can think of, is that Apple thinks I decided to unofficially deploy the app with TestFlight without going through the formal review. I wish I could talk to a human so they could comfirm this is the problem, but all I get are bots, bots, and more bots. If that is the reason why Apple wants to terminate my entire developer account, then perhaps removing the app from TestFlight would have sufficed? I do not know. In a sea of automated actions it is impossible to find anything that make sense.

# Where I Stand Now

Apple has taken no action on my other apps that never hit the App Store. I have a total of 8 apps on my account. Two were published a long time ago (2011, 2012) and I removed them because they don't perform well anymore. The others are apps I worked on but never had the drive to finish - I will be happy to remove these if Apple is thinking that I got them to squat their names. Out all the apps I have there, one is on TestFlight (Mignori), and the other has been on the App Store since 2017 (Next Anime Episode). My App Store app is still downloadable. People can still search for it and download it. And people can still review it. In the last few days, Next Anime Episode has started to receive an unusual number of 1-Star reviews and all in a quick succession, so I am starting to think that someone is targetting me for god knows what reason. I can respond to reviews, and I responded to one today, but Apple has not approved it, and I do not know if they will.

Just yesterday I uploaded a new version of Next Anime Episode with a small bugfix to see if I could still do everything on the Apple Developer Program, and I can. The upload went fine, although it has been waiting for review for almost 24 hours now. I do not know if they haven't reviewed it because they working slower at reviews with WWDC around the corner or because they don't take a look at Flagged apps.

One thing I find particularly weird is that I know Apple puts accounts "under investigation" for reasons unknown to the developers before they terminate them. Apple never put my account under investigation before telling me they are going to terminate my account from the program. This came out completely out of the blue. In fact, on June 8 of this year, a little bit over a week before Apple acted against Mignori, I submitted an update for Next Anime Episode and the review went by fast and smoothly. I know Apple doesn't review apps of accounts under investigation, so I know I wasn't being investigated.

I know of developers who were instantly terminated and all their apps removed. Luckily this has not happened to me. I can still view my payments (I am 8 dolars away from getting my last $100 payment, and I hope I get them before my account is terminated because that way I can recoup the cost of the developer program I JUST RENEWED last week), and I can still do everything. I was even able to accept the latest developer agreement for paid apps.

I know people don't care about a small developer's feelings, especially Apple, but that won't stop me from saying I'm feeling incredibly bummed about this. I have been part of the Developer Program since 2011. I have been active on the iOS community since then. I started to blog about iOS in 2011, and I revised my blog last year to re-launch it and blog more often than ever. I have managed to write one article per week because I *love* the iOS community. Part of the reason I blog is because I can learn while I blog and that helps me become a more competent developer. I now I am not a popular developer, but I still want to say that I pour a lot of effort into what I do because I genuinely like(d) all this, and the iOS developer community is one of my favorite communities.

I am bummed, because I have been making a living out of iOS development for 9 years and everything is suddenly coming to an end without me being able to talk to a human at Apple. All the skills I have built over the course of almost a decade are obsolete now, because I do not imagine myself working for other people as an iOS dev without having my own hobby iOS apps on the App Store. It feels like all the blogging I have been doing has been a waste because I cannot participate in the knowledge I myself write, and it's pointless to engage in the knowledge of others if I cannot try what they write.

If by any chance this gets far enough for someone at Apple to see it... Please, reach out. Get a human to send me an e-mail or make a call. I won't even ask you to avoid terminating my developer account if you can tell me what, exactly, I did wrong. Mignori was never on the App Store. It has been on TestFlight all this time. All my other apps are fine. All I want is to talk to a human to understand where things went south.