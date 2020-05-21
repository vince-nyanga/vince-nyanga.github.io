---
title: "State management in Flutter: The BLoC pattern"
date: 2020-01-25
categories: [Flutter, Mobile]
---

In every application state management is one of the most important things you need to deal with as a developer. You need to properly manage state in your application to ensure that your users are not frastrated and also to avoid maintenance headaches. In Flutter the need to have proper state management is no less important. In this post I am going to talk about one of the patterns you can use to manage your application state -- the BLoC pattern. This is going to be an overview of the pattern; in the next post I am going to illustrate how the pattern works buy building a simple authentication flow that uses the pattern.

## What is BLoC

BLoC stands for **B**usiness **Lo**gic **C**omponent. Like the name suggests a BLoC is a business logic layer that sits between the user interface and the data source(s). It seperates the presentation layer from the business logic almost similar to the MVC and MVVM patterns. It makes use of reactive programming to handle the flow of data from the data source(s) to the UI and vice versa.

## Core components of the BLoC Pattern

There are 3 main components in the BLoC pattern.

### 1. Events

Events are the inputs to the BLoC usually in response to user interaction. For example, an event can be a button click. When a user clicks a button an event is sent to the BLoC carrying information about the intensions of the user. Here's an example of an event:

```java
class LoginWithEmailButtonPressed extends LoginEvent {

    // The data stored in the event.
    final String email;
    final String password;

    LoginWithEmailButtonPressed({
        @required this.email,
        @required this.password});
}
```

The event above is sent to the BLoC when a user fills in the login form and presses the login button. The BLoC will receive this event, process it accordingly and send the data to the authentication service.

### 2. States

States are the output of the BLoC which represent a part of the application state. The UI layers 'listens' for changes to these states and redraw themselves. Flutter uses a declarative way of building UI and widgets are immutable. This means if there is a change in state a new widget needs to be built to represent the new state.

Using our login example we can have these states: `LoginIdle`,`LoginLoading`, `LoginSuccess` and `LoginFailure`. When the UI sends the `LoginWithEmailButtonPressed` event to the BLoC, the BLoC changes the state from `LoginIdle` to `LoginLoading` state while waiting for the response from the authentication service. The UI will get notified of the change in state and shows a progress indicator to the user. When the BLoC receives a response from the authentication service it changes the state to `LoginSuccess` or `LoginFailure` depending on the response. Again, the UI will get notified of the change and redraws itself accordingly -- take the user to the home page if it's a `LoginSuccess` state or show an error message if it's a `LoginFailure` state.

### 3. Streams

Those who have done some reactive programming should know streams. Streams provide a sequence of asynchronous data. Take for example a pipe of water. The pipe is the stream and the water flowing through the pipe is the asynchronous data. A component can 'plug into' to the stream to get the asynchronous data as it comes in.

In the BLoC pattern there is a stream of `Event`s from the presentation layer and a stream of `State`s from the BLoC. The BLoC is there to convert a stream of incoming events into a stream of outgoing states.

In our login example the BLoC 'plugs' into the `LoginEvent` stream from the UI. The UI 'plugs' into `LoginState` stream from the BLoC. Here's how data will flow:

1.  When the user presses the login button, the UI pushes the `LoginWithEmailButtonPressed` event into the `LoginEvent` stream.
2.  The BLoC will receive the new event, send the data from the event to the authentication layer and push a `LoginLoading` state into the `LoginState` stream while waiting for a response from the authentication layer.
3.  The UI receives the `LoginLoading` from the `LoginState` stream and displays a progress indicator.
4.  When the BLoC receives a response from the authentication service, it pushes the appropriate state -- `LoginSuccess` or `LoginFailure` into the `LoginState` stream.
5.  The UI will recieve the new state and redraws itself accordingly.

## Summary

The BLoC pattern is a state management pattern that makes use of reactive programming. At the core of the pattern are `Event`s, which are sent from the presentation layer in response to user interaction; `State`s which represent the application state and come from the business logic component (BLoC) and `Stream`s which are sequences of asynchronous data. Below is a simple illustration of how the pattern works.

<figure>
<img src="{{ site.baseurl }}/images/flutter/bloc/bloc.png" alt="BLoC pattern">
<figcaption>Illustration of the BLoC pattern</figcaption>
</figure>
In the image above components on the left hand side do not know anything about those on the right hand side. The data repository doesn't know anything about the BLoC layer and the BLoC layer doesn't know anything about the UI layer. This dependency inversion makes testing and maintenance of the code very easy.

Now that you know how the BLoC pattern works we will put it into practice in the next post when we build an authentication flow in Flutter. Thanks again for reading and hopefully you have learned something.
