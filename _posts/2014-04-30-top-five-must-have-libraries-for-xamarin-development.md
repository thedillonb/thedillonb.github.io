---
layout: post
published: true
title: Top Five Must Have Libraries for Xamarin Development
categories: Programming
tags: 
  - xamarin
  - mobile
---

I create a lot of Xamarin projects in my free time. Each one a little different than the last and, consequently, a little better developed. Below is a list of five libraries I find to be the most benefical to developing Xamarin iOS / Android mobile applications. 

## 1) [ModernHttpClient](https://github.com/paulcbetts/ModernHttpClient)
If your app does any sort of network access this library will change your world. It's sole purpose is to make your network IO go _drastically_ faster. It does this by calls into native platform libraries like NSUrlSession or OkHttp 1.5 for iOS and Android, respectively. It's only ask is that you use HttpClient and instantiate the object with either an iOS or Android handler - which will make the native calls underneight.

## 2) [Akavache](https://github.com/akavache/Akavache)
While ModernHttpClient will make your network IO faster, it's best to never make the calls in the first place if you don't have to - which is where Akavache comes in. Akavache is like memcached for mobile applications. While it can technically store anything from arbritrary objects to secure credentials, I find it best served as a network cache for any type of request, even images. 

## 3) [MvvmCross](https://github.com/MvvmCross/MvvmCross)
If you expect to run your application on more than one platform then you should be using MvvmCross. MvvmCross is a library for implementing the [MVVM](http://en.wikipedia.org/wiki/Model_View_ViewModel) design pattern in mobile applications so that "core" code (everything but visual stuff) can be re-used on different platforms. MvvmCross is extremely well documented and provides dozesns of examples and videos to learn. MvvmCross is an all-in-one package (with plugin support) that makes cross-platform development stupidly easy.

## 4) [Reactive Extensions](https://rx.codeplex.com/)
Reactive extensions is the bees knees. It's the best thing to happen to event driven code in a long time. The number of cool things you can do with Reactive Extensions is mind boggling. If your application works with events I __highly__ recommend a look into Reactive Extensions. Reactive Extensions basically turns events into objects that can be used in LINQ statements. If that hurt your brain, like it did when I first looked at Rx, then check out [Intro To Rx](http://www.introtorx.com/) for more details and examples.

## 5) [ReactiveUI](http://www.reactiveui.net/)
Like MvvmCross, ReactiveUI is another MVVM design library with one exception: it is centered around Reactive Extensions while the other is a more conventional MVVM style. Think of ReactiveUI as more of a "pick and choose" framework while MvvmCross is more of a "full MVVM experience". Regardless, if you like Reactive Extensions, and are going for a MVVM type project, then ReactiveUI is definitely worth your time.

## Bonus: [Parse](https://parse.com/)
This one is not really a library but then again, this is not in the 1 to 5 list. If you need cloud support, like analytics, push notifications, or data storage, than you should definitely check out Parse. Their __free__ package is crazy generous. They have a wonderfully simple and light weight assembly for Xamarin projects that make communication simple. I am in no way affiliated with them but their offering is too good to not look into.