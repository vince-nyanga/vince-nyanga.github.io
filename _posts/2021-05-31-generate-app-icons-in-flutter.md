---
title: "Generate App Icons In Flutter"
date: 2021-05-31
tags: [Flutter]
---

In this post I am going to talk about how you can generate app launcher icons in Flutter using a wonderful package that I stumbled upon two years ago when I started building Flutter apps. This is going to be a very short read but I figured someone out there might be scratching their head over this very issue. As we all know, your app icon needs to support different screen sizes and resolutions. It would be cumbersome if you had to manually resize your icon to cater for all screen densities on both Android and iOS.

This is where the [flutter_launcher_icons](https://pub.dev/packages/flutter_launcher_icons) package comes in handy. It is a flexible command line tool that helps update your app icons, ensuring that all screen sizes and densities are taken care of for both Android and iOS. So how does it work? Firstly, you and graphics team create a beautiful icon of course. Add it to a directory in your application, say `assets/images/`. After that add the `flutter_launcher_icons` package to your `pubspec.yaml` under `dev_dependencies` or you can create a separate config file:

```yaml
dev_dependencies:
  flutter_launcher_icons: "^[0.9.0]"
```

Add configuration for the package:

```yaml
android: "launcher_icon" # you can specify the name of the icon
ios: true # if false then it won't update for platform
image_path: "assets/images/icon.png"
```

Then run the package:

```bash
flutter pub get
flutter pub run flutter_launcher_icons:main
```

If you have a separate config file then you will need to specify its location when you run the package. Make sure your config file is in the same directory as your `pubspec.yaml`:

```bash
flutter pub get
flutter pub run flutter_launcher_icons:main -f [path-to-config-file]
```

This is just the basic setup. There are many attributes that you can set to further customise the package. For more detailed information check out this [site](https://pub.dev/packages/flutter_launcher_icons).

Thanks for taking your time to read. Hopefully it was helpful.
