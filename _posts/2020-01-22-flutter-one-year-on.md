---
title: "My Flutter Journey: One Year On"
date: 2020-01-22
categories: [Mobile, Flutter]
---

Around August 2018 I came across Flutter, a multi-platform toolkit for building mobile apps, desktop apps as well as web apps. It sounded very interesting and I told myself that I needed to learn it some day. I did a `Hello World` project and forgot about it for months.

Fast forward to five months later, I had a mobile app project I needed to work on. I needed to have both iOS and Android versions of the app and as an Android developer with minimal knowledge of `Swift`, I needed to find a way of building an iOS version of the app without having to have two seperate projects. I remembered that I had come across a multi-platform toolkit a couple of months earlier and decided to revisit it and see if I could use it in my project. After reading the documentation and doing a few code labs I decided to adopt Flutter for my project and the rest, like they say, is history. In this post I want to talk about my experiences with Flutter over the past year or so -- what I enjoyed and the struggles I encountered.

## What is Flutter

To begin let us get a brief overview of what Flutter is. According to their [website](https://flutter.dev/), Flutter is Google's UI toolkit for building beautiful, natively compiled applications for mobile, desktop and web from a single codebase. With Flutter one code base can be compiled into native iOS, Android, desktop and web apps. This is exactly what I wanted so I decided to use in in my project.

## What I enjoy about Flutter

Since I started working with Flutter I seldom work with Kotlin to build my apps. This is not because there is something wrong with Kotlin but there are certain things that I like about Flutter and here they are, among others:

### You can build almost anything

In Flutter everything is a widget. This makes it easy to build custom widgets of your own or modify existing ones. I found out that bringing designs into life is very easy with Flutter and everything looked beautiful; animations were also very smooth.

### Hot reload

Flutter has a hot reload feature that allows you to quickly experiment, build UIs and fix bugs. This came in handy for me since I was both the designer and the developer of the app. It gave me the ability to change designs and almost instantaniously see the result on the screen. If you want to learn more about this feature check out this [link](https://flutter.dev/docs/development/tools/hot-reload).

### It's easy to learn

I found `Dart`, the programming language used to build Flutter apps, very easy to learn. In addition Flutter's extensive documentation allowed me to find solutions to problems I encountered on the way. Since I was learning the framework as I was developing the app I hardly encountered problems that took me long to resolve. In addition, Flutter has a growing community from which one can easily find help.

## What I struggled with

Even though Flutter had so many things that I enjoyed, I struggled initially with state management. Flutter does not have out-of-the box state management (at the time of writing) which means you need to find the best way to manage state inside your app. Initially I went with Inherited Widgets and a couple of months after releasing the Android version of the app I re-wrote everything using `BLoC/Rx` which seemed to fit my app very well. For more information on state management check out this [link](https://flutter.dev/docs/development/data-and-backend/state-mgmt/options).

## What happened to the project

It took me about 3 months working on the project part-time (after work and during weekends) to have something that was usable. When I work on side projects I have a simple philosophy -- make it work first then make it fast and pretty later. In the second week of April 2019 I released version 1.0.0 of the [Android](https://play.google.com/store/apps/details?id=me.vincenyanga.re5) app. In July/August of the same year I re-wrote the entire app with a different architecture.

Even though I could build both iOS and Android apps from the same code base, I decided to make different UIs for the platforms so the app looks 'iOSy' to iOS users and 'Androidy' to Android users. I started working on seperating the user interface from the core application code around September 2019. A month later I released the initial version of the [iOS](https://apps.apple.com/us/app/re-5-made-easy/id1483182746) app that shares everything with the Android app except the UI that uses Cupertino widgets. There are some widgets that are common in both platforms like the `QuestionView` that renders the question on the screen. Here's how the apps look at the time of writing:

<figure class="half">
<img src="{{ site.baseurl }}/images/flutter/journey/android.png" alt="Android app">
<img src="{{ site.baseurl }}/images/flutter/journey/ios.png" alt="iOS app">
<figcaption>RE 5 Made Easy. The Android app on the left uses the Material design widgets while the iOS app on the right uses Cupertino widgets.</figcaption>
</figure>

## Conclusion

In this post I spoke about my experiences with Flutter -- things that I enjoyed as well as things that I struggled with. This is the begining of many posts on Flutter where I will be sharing what I have learned while working with the framework. I'm by no means an expert in Flutter but everyday I try to hone my skills more and more.

I highly recommend that you take a look at Flutter if you want to build natively compiled applications for Android and iOS from one codebase. Visit their [website](https://flutter.dev/) to get started.
