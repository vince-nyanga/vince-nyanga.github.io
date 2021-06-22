---
title: "How Dare You Kill My Foreground Service"
date: 2021-06-16
tags: [Android]
---

_Edit -- 22 June 2021: After more tests we discovered that the problem devices worked fine when the phone was charging which means we're affected by the [Doze Mode](https://developer.android.com/training/monitoring-device-state/doze-standby). This still leaves me with more questions than answers since nothing seems to work on these devices including changing battery optimisation settings. I have since decided to ping the device from the backend using FCM whenever I discover that I'm no longer getting location from it._

Allow me to express my frustration about a challenge that I'm facing with Android, well some Android device manufacturers actually. I have been consumed by this problem for the past couple of months and no matter what I try I don't seem to be cracking it. In this article I'm going to talk about how certain device manufacturers are allegedly not playing by the rules or have special rules of their own that stock Android doesn't have, making it incredibly hard for developers to be certain that what they write will work on all devices out there.

Let me summarise the problem I'm facing. In my app I want to be able to get the user's location at a fairly regular interval. The location should keep coming whether the app is in the foreground or background. This is a classic case where a [foreground Service](https://developer.android.com/guide/components/foreground-services) shines.

For the benefit of those who might not have encountered foreground `Service`s in Android, it is a [Service](https://developer.android.com/guide/components/services) that performs an operation that is noticeable to the user. In order for the user to know that something is going on, it is mandatory that a foreground `Service` displays a notification for as long as it's running. What sets it apart from the regular `Service` is that it will continue to run even when the user is not interacting with the app. The Android OS tries by all means not to 'kill' a foreground `Service` save for extreme circumstances. For example, when a music player app is playing music, the music will continue playing even when you're not in the app itself. You see a nice notification in the notification tray telling you what song is playing.

Why does the OS try not to 'kill' foreground `Services` you may ask. Let me answer by talking about processes in Android. Almost every Android application runs in it's own Linux process. The process begins when you start the application and it will continue to run until you stop your application, or when the system wants to reclaim some memory so other apps may run. Not all processes are equal in Android. When the OS wants to decide which process to terminate it uses a hierarchy based on what components of your application are running and in what state they are in. I'm not going to talk too much about it but just know that a foreground `Service` is placed in the second most important process type which means it's highly unlikely that it will get 'killed' by the OS -- in theory at least. If you want in-depth information on this please visit the [android docs](https://developer.android.com/guide/components/activities/process-lifecycle).

Given the information above, I thought the best approach to ensure that I get the user's location at regular intervals regardless of whether or not the app is in the foreground, was to use a foreground `Service`. I followed all the recommendations, registered my `Service` with foreground type `location`, ensured I requested the `FOREGROUND_SERVICE` permission. I also made sure the `Service` is [_sticky_](https://developer.android.com/reference/android/app/Service#START_STICKY) so it can be restarted by the OS when there are enough resources for it to run:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" ...>
    <!-- I requested FOREGROUND_SERVICE permission -->
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>

    <application ...>
       <!-- I also set foreground service type to location and dataSync-->
       <service ... android:foregroundServiceType="location|dataSync"/>
    </application>
</manifest>
```

Inside my `Service` I use the [FusedLocationProviderClient](https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderClient)'s `requestLocationUpdates()` function to get the location at regular intervals. This works very well, at least that's what I thought until I discovered that's not the case on certain devices -- **W**ell, **T**hat's **F**antastic!

After digging through the internet to try and understand what's going on I discovered that some manufacturers, in a quest to ensure long battery life, have implemented ruthless processes that kill everything with impunity as soon as the app goes to the background or when the device is locked-- see [this site](https://dontkillmyapp.com/). But I have a foreground `Service` so my app's process shouldn't be terminated right? Wrong! Some of them don't care. They will murder it in cold blood.

What should I do then? It turns out there are battery optimisation settings that the user needs to change for your app to run properly in the background. This would be easy if the settings were standard on all devices so I can help my users change them but no, each manufacturer has their own way of doing it. It looks like the onus is on the user to make changes that will guarantee that my app works as it should. That's not ideal. On certain devices no matter how many settings I change as soon as the device screen is locked location updates stop.

One of the manufacturers apparently has a whitelist of apps that are allowed to run in the background without problems -- The Immortals. How do I get my app into the list? I have no idea.

This whole situation is frustrating the life out of me. I did everything (at least I think so) the Android documentation says I should do. However, some manufacturers seem to ignore the contract and now I'm faced with an app that's not behaving as it should on certain devices and I have no way of programmatically getting around the problem except for me to nicely ask my users to go change their settings, which doesn't guarantee success.

I don't have any words left except to ask them **How dare you kill my foreground service?** when the documentation says it should not be killed that easily.

### Disclaimer

- Foreground `Service`s may be terminated by the OS if doing so is required to keep all foreground processes running. This should be an exception not a norm.
- I might have missed something in my implementation. I will keep on searching for possible solutions. If you have an answer your help will be greatly appreciated.
