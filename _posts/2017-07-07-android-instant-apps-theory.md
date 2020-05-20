---
title: Android Instant Apps - theory
date: 2017-07-07
tags: [Android]
---

As you may know Android Instant Apps are now open to all developers. This is the news people like me have been waiting for for a very long time. In this post I am going to talk about the theory behind Android Instant Apps as I understand it.

### What is an Android Instant App?

What is an Android Instant App?
An Android Instant App is an application that can run on a device without the user having to install it first. It is launched in response to a user launching a URL. What basically happens is that the user launches a URL and instead of the phone opening the URL in a browser, a native Android app is launched and the user can use it enjoying all the features of a native app. For instance, a user opens a link https://my-mock-re-app/practice. If there is an Instant App on the app store for that URL and the user’s device supports Instant Apps then the Instant App is launched and the user can start a practice session. This has the following advantages, among others:

- Users can have a look at the features of an app before they make a decision to install it.
- Users can share stuff from the app with their friends and they (friends) can enjoy the same experience without the need to have the app installed on their device.

Android Instant Apps provides users with a great experience.

### What happens under the hood?

So what happens when a user launches a URL on their device? When the user launches a URL Google Play checks to see if there is an instant app matching that URL. If there is an instant app matching Google Play then sends the necessary code files to the device so the device can run the app. The flow chart below shows the process as I understand it.

<figure>
<img src="{{ site.baseurl }}/images/instant-app.png" alt="Instant app flowchart">
<figcaption>A flowchart that shows how Instant apps work</figcaption>
</figure>

### Features as building blocks

An Instant App is a made up of features. A feature is module that applies `com.android.feature` and behaves differently in different circumstances:

- When it’s consumed by a normal application (`com.android.application`) during the build process, it produces an `aar` file and works just like any other library.
- When it’s consumed by an Instant App it generates an Instant app APK.

For instance, a map app can allow users to navigate, search for nearby points of interest or share current location. All these actions are features that make up the app. Now let’s say, for example, the user launches this URL: https://instant-app.com/navigate/. The navigation feature is sent to the device and the user will start navigating to their destination. The same applies for https://instant-app.com/search/ and https://instant-app.com/share/.

### Base Feature

Each Instant App must have **one** (and only one) base feature. This feature is sent to the device with the feature requested by the user. It should contain all the shared resources and code that all the other features use like styles. In our map app example, the Instant app will consist of four features — _base-feature_, _navigation-feature_, _search-feature_ and _share-feature_. So when the user requests the navigation feature APK two feature APKs are sent; the base feature and the navigation feature.

### What else to know about features

The following are what developers need to know when they create features for their apps:

- A feature APK should be less than 4MB so you should make them as small as possible
- A feature should have at least one Activity
- An Activity in a feature cannot launch another Activity. Instead it has to request a URL corresponding to that Activity
- Instant App cannot launch Activity in another feature. The same process as above has to be taken

### Summary

In this post I spoke about Android Instant Apps — what they are as well as how they work. In the next post we are gonna convert an existing app to an Instant App and maybe create one from scratch.
