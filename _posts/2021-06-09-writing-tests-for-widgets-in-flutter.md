---
title: "Writing Tests For Widgets In Flutter"
date: 2021-06-09
tags: [Flutter, Testing]
---

Writing tests for your software is one of those things every software developer can't argue against (a few still do though). There are a ton of benefits you get when your software is sufficiently covered by tests, be it unit tests, integration tests, or any other test you know. This truth still holds when it comes to Flutter. When you are building Flutter apps it is very important that you thoroughly test them. In this post I am going to talk briefly about how you can write tests for your widgets in Flutter.

As you know by now, everything is a widget in Flutter. When you create a login screen, it's just a bunch of widgets that you compose together and add some logic to handle the user's input. The Flutter SDK provides a `WidgetTester` which allows you to build and interact with your widgets in a test environment. This wonderful class is part of the `flutter_test` package that comes bundled in the SDK. All you need to do is to add it in your `pubspec.yaml` and you're ready to go:

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
```

There are a few steps that you need to follow when you are writing tests for your widgets. I'm going to summarize them but if you need to get more in-depth knowledge I suggest you check out this [cookbook](https://flutter.dev/docs/cookbook/testing/widget/introduction) from Flutter.

1. Begin your test by calling the `testWidgets()` function. This function allows you to define a widget test and create a `WidgetTester` that you will use to interact with your widget under test:

   ```dart
   testWidgets('What you are testing', (WidgetTester tester) async {
       // test your widget here
   });
   ```

2. Use the `WidgetTester` to build the widget under test. The `WidgetTester` has a function `pumpWidget()` which builds and renders the widget that you are testing.

   ```dart
   testWidgets('Should contain welcome message', (WidgetTester tester) async {
       await tester.pumpWidget(MyApp());

       // your widget has been built and rendered.
       // you may now interact with it.
   });
   ```

3. Use the [Finder](https://api.flutter.dev/flutter/flutter_test/Finder-class.html) class which searches your widget tree and returns a node that matches the criteria you provide to get the node you want to test. The `Finder`. `flutter_test` provides a top-level function `find()` that instantiates and uses the [CommonFinders](https://api.flutter.dev/flutter/flutter_test/CommonFinders-class.html) which provides a lightweight syntax for accessing commonly used `Finder`s. There are many criteria you can use to search for the node you want. Visit the [CommonFinders](https://api.flutter.dev/flutter/flutter_test/CommonFinders-class.html) docs for more information.

   ```dart
   testWidgets('Should contain welcome message', (WidgetTester tester) async {
       await tester.pumpWidget(MyApp());

       final welcomeTextFinder = find.text('Welcome');
       // ...
   });
   ```

4. Next you interact with your widget. This is an optional step as some widgets such as `Text` won't need you to interact with them. This is is where you use the `WidgetTester` to programmatically interact with your widgets using the [WidgetController](https://api.flutter.dev/flutter/flutter_test/WidgetController-class.html) class. The `WidgetController` provides a bunch of actions you may take on your widgets -- tapping, dragging and entering text, among others. After you have completed your interactions don't forget to call the [pump()](https://api.flutter.dev/flutter/flutter_test/WidgetTester/pump.html) function -- very important. The `pump()` function schedules a frame and triggers a rebuild of your widget under test. If your interaction involves animations you might want to call the [pumpAndSettle()](https://api.flutter.dev/flutter/flutter_test/WidgetTester/pumpAndSettle.html) function which repeatedly calls `pump()` until all animations are completed.

   ```dart
   testWidgets('Should enter and display name', (WidgetTester tester) async {
       await tester.pumpWidget(MyApp());

       String name = 'Vince';

       // confirm non-existance
       expect(find.text(name), findsNothing);

       // use the WidgetController to enter text
       await tester.enterText(find.byType(TextField), name);

       // use the WidgetController to tap on a button
       await tester.tap(find.byType(ElevatedButton));

       // Schedule a frame and force rebuild
       await tester.pump();

       // Find and verify (step 5)
       expect(find.text(name), findsOneWidget);
   });
   ```

5. Use the `Matcher` to verify your widget. The `flutter_test` library provides top-level constants for commonly used `Matcher`s such as `findsNothing`, `findsOneWidget`, `findsWidgets`. For more information check out the [docs](https://api.flutter.dev/flutter/flutter_test/flutter_test-library.html).

   ````dart
    testWidgets('Should contain welcome message', (WidgetTester tester) async {
        await tester.pumpWidget(MyApp());

        final welcomeTextFinder = find.text('Welcome');

        expect(welcomeTextFinder, findsOneWidget);
    });
    ```
   And you are done. These are the 5 steps (well, my 5 steps) that you take when you are writing tests for your widgets. I will need to stress this important point again -- don't forget to `pump()` after you've interacted with your widget(s). Trust me, it will save you a lot of headaches.
   ````

Once again, thank you for taking your time to read and I hope you have learnt something. Personally, writing this post gave me the opportunity to dig deeper into the documentation to uncover more things I didn't know which is great. Till next time, have a great one!
