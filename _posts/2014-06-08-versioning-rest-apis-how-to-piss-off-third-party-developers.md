---
layout: post
published: false
title: "Versioning REST APIs: How to Piss Off Third Party Developers"
categories: Programming
tags: 
  - programming
---

The concept of versioning a RESTful API is very basic: you start off developing a set of APIs which you'll label version 1.0. A few months later, you choose to make modifications to the existing set of APIs. Instead of modifying the current structure you choose to make a 2.0 verison. The whole purpose of this process is that you prevent disruption for those using your 1.0 API while you work on a new set of features. If you choose to version your API but ignore this fact and butcher your 1.0 anyway, while not providing a suitable replacement in your 2.0, then you might as well tell your third party developers to get lost.

There is a company, which I have developed a mobile app for, which is a constant offender of this golen rule. I'll call this company Telamericorp for the sake of ananimity. Telamericorp is not the biggest fish in the sea but it has great potential. It's got a good product and a set of APIs which allow third party developers to begin designing applications for. For a budding product this is the dream: to have third party developers creating applications which help promote and greater enable the use their service.

Telamericorp uses a versioned API to enable new features and repair existing ones as they constantly evolve. Developers use the stable API and design applications that go out to thousands of users to promote use of their application as well as Telamericorps. Everyone wins! Everything is running smoothly until one day third party applications stop reporting a peice of data that was once there. Chaos arises and the third party developer goes into a panic.

If you're a third party developer then you'll probably do the following:

1. File a ticket with Telamericorp
2. Respond to pissed user emails 
3. Begin a work around - if one exists

All the while you'll wonder to yourself, "I used a stable API version... Why in God's name did they make modifications to it and not just alter the 2.0 version!" Personally, when this happens, it makes **me** look bad even though I didn't cause the issue! Which then leads me to rethink developing applications for Telamericorp.

> Mismanaging your versioned API is a great way to piss off developers, kill your third party ecosystem, and make sure your product receives no developer evangelism.

If you don't care about third party applications then don't even bother with a versioned API system - it might even slow you down. However, if third party applications are a key component to your product's success then a versioned API is the way to go as long as you **don't change anything within a stable API version!**



