---
title: "What I Learned From Selling an App"
date: 2021-03-03T07:00:00-04:00
originalDate: 2021-02-27T18:24:52-04:00
publishDate: 2021-03-03T07:00:00-04:00
draft: false
highlightjslanguages:
 - swift
 - objectivec
tags:
 - apple
 - app transfer
categories:
 - development
keywords:
 - apple
 - app transfer
description: "The lessons I learned from transfering my app to another developer."
---

What I Learned From Selling an App

A few weeks ago, I sold and transferred my app, [Next Anime Episode](https://www.andyibanez.com/projects/nextanimeepisode1/), to another developer.

In this article, I will discuss the reasons I transferred my app, and what I learned along the way in the process of transferring it.

# Why I Sold My App

As a developer who works a full time job and work on indie apps as a hobby, there are many ideas I want to work on. Having apps on the App Store is actually a really big responsibility. You have to maintain it and add features at least every so often to it in order to keep interested. If the app becomes popular enough, the pressure just grows, because you have to start dealing with negative reviews of all kinds. Sometimes justified, sometimes people don't understand the purpose of your app, sometimes they just insult it and give no feedback whatsoever to improve it.

The mental strain grows.

If you are an indie dev with an app on the App Store, you know there's a lot of implicit responsibility you have to take upon your shoulders. And eventually, you may come to realize you are just not as passionate about that project as you used to be when you first started it. You want to move on to other projects, and keep working on you career. It happens to all of us, and sometimes, we really just want to move on.

I got lucky because I wasn't actively trying to sell my app, but the offer landed on my lap and I heavily considered it. I pondered for weeks and I realized the best course of action was to hand over the app to someone who was going to maintain more often than me, fix bugs, add features, and overall, give users the treatment they deserve. I was only releasing about one update every 8 months on the app, and I sincerely thought users deserved better, so I handed it over.

# Goodbye, Small Developer Program

My app wasn't a one-million dollar idea. If anything I got much, *much* less than that. I am not going to disclosure the full amount, but I was going to make the amount I got from that sale in about five to seven years at the rate it was picking up this year. If you are an indie, you know we need to make $150 to get paid, and my app barely made one payment each year, so you can imagine that my sale wasn't exactly gigantic.

Despite how small the sale is, I got removed from the Small Business Problem as one of the conditions for staying on it are to not transfer an app. All my sales are going to give Apple a 30% commission for the rest of the year. I am hoping I will be able to apply to the program again so I can go back to the 15% rate (which I wasn't able to enjoy).

# Not Keeping Analytics is a Double-Edged Sword.

Because of all the privacy rules that are popping up all over the world (starting by the GRDP), and the fact I am a small indie developer as a hobby and not a lawyer, I take measures in my app to prevent getting into legal trouble for collecting data.

So first, I outright do not collect any data. None of my apps currently use any analytics frameworks of any kind. Despite how useful they are and they can be used without leaking user data, I decided to never use them.

These had two important implications in the sale of my app:

1. Because I do not collect any data on that app whatsoever, I did not have to do anything regarding disclosure of data transferring hands. Depending on the jurisdictions of your users, if their data is going to a new company, you are legally required to disclose that. My app interacted with an API, but there were never user accounts involved, and it simply performed HTTP requests. Not having any user data made the transfer easier in a way, so much to the point that, if my app was an app with user accounts and it did store user data in a server, I would not have sold it simply because I am not a lawyer and I do not know how to handle personal data switching hands. Before you transfer an app, you are required to remove all TestFlight testers, so the new owners do not even get that.
2. This is the other side of the coin, but also not having analytics made the sale of the app a bit more complicated. When the potential buyers were appraising the app, they asked a lot of questions regarding analytics frameworks and the data I had. They wanted to have more detailed information over what Apple provides. Despite that, my app was climbing the ranks on its own, so it was clear to see it was doing well without any kind of analytics framework.

# Not Using Paid Services Makes Transfers Easier

My app used a public API and nothing more. In the earlier versions, my app used a server with custom software in order to send notifications to users. After the API I was working with was upgraded to v2, their API started returning more data and the data I had was enough to queue the notifications locally. Not using any paid Push Notification Services helped me sell the app as that did not represent any ongoing costs to the new developers.

# No Contracts Involved

I have never sold an app before and I do not know how common it is these days, but I sold the app under no contracts whatsoever. Both the buyers and myself followed a series of steps to ensure we would all get what we wanted:

1. I would transfer the app from App Store Connect.
2. After transferring the app, they would make the deposit (international wire transfer) into my bank account.
3. After I received the deposit, I sent the source code to the new developers.

And that was it. If you have enough failsafes you can transfer your app risk-free. The people I sold the app to were trustworthy, but in retrospect, since there were no contracts involved, it probably would have been a good idea to add some sort of kill switch into the app in case they didn't transfer the money and I never sent the source code. It's probably better to sign some sort of contract, but my own experience was smooth.

# You Will Always Receive Lowball Offers

Be certain of the value of your app. Many offers can be lowball and if you are certain of the value of your product you can easily get much more than the original offer.

# Everything for the Road Ahead

I have never sold an app before, and it did felt a little bit weird, perhaps nostalgic, having to sold something you wrote yourself like this. When I communicated these to my users, some told me the news were bittersweet, but overall they were excited I was going to be able to dedicate more times to other projects.

On the other hand, I am really happy I built something that someone else was interested in buying. It made me realize that what I was doing was really worth it, and I'm happy I was able to finish that sale.
