---
title: "Building an authentication flow in Flutter using the BLoC pattern"
date: 2020-02-01
tags: [Flutter, Mobile]
---

In the [previous post]({{ site.baseurl }}/state-management-in-flutter-bloc-pattern/) we introduced the `BLoC` pattern as one of the state management solutions in Flutter. In this post we are going to put that theory into practice by building a simple authentication flow that utilises the pattern. This is going to be a simple Flutter app that has three screens -- a splash screen, a login screen and a home screen.

**Update: 12/10/2020:** Updated to `flutter_bloc: ^6.0.6`

## App Overview

Before we get into the code let us get a brief overview of how the app is going to behave. This is how the app is going to behave:

- When a user opens then app a splash screen is displayed while the authentication service is checking if user is authenticated.
- If the user is not authenticated they are redirected to the login screen otherwise they will go to the home screen.
- If login is successful the user is redirected to the home screen otherwise an error message is displayed.

## The Code

I am assuming you have Flutter installed on your machine and you know how to create a flutter app. If you do not know then you need to visit the Flutter [website](https://flutter.dev/) to get started. We are going to use the [flutter_bloc](https://pub.dev/packages/flutter_bloc) library -- a library that helps implement the BLoC pattern. Create a new application and add the following dependencies to your `pubsec.yaml`:

```yaml
# pubsec.yaml
# ...
dependencies:
  bloc: ^6.0.3
  flutter_bloc: ^6.0.6
  equatable: ^1.0.2
# ...
```

For brevity not all of the code will be included in this post. Things like models and services can be found on [GitHub](https://github.com/vince-nyanga/flutter_bloc_authentication/tree/master/lib). Our focus is going to be on the `BLoC` and user interface. Let's get started.

### `BLoC`s

We are going to have two `BLoC`s with their associated events and states -- `AuthenticationBloc` and `LoginBloc`. Create a new directory inside your `lib` directory and name it `blocs`. Let us start with the `AuthenticationBloc`, its events and states.

#### Authentication

Inside the `blocs` directory create another directory and name it `authentication` then create a new file `authentication_event.dart` and add the following code therein:

```java
// lib/blocs/authentication/authentication_event.dart

import 'package:meta/meta.dart';
import 'package:equatable/equatable.dart';

import '../../models/models.dart';

abstract class AuthenticationEvent extends Equatable {
  const AuthenticationEvent();

  @override
  List<Object> get props => [];
}

// Fired just after the app is launched
class AppLoaded extends AuthenticationEvent {}

// Fired when a user has successfully logged in
class UserLoggedIn extends AuthenticationEvent {
  final User user;

  UserLoggedIn({@required this.user});

  @override
  List<Object> get props => [user];
}

// Fired when the user has logged out
class UserLoggedOut extends AuthenticationEvent {}

```

These are the events whose stream the authentication `BLoC` will listen to. While you are still in the same directory create another file, `authentication_state.dart` and add the following code:

```java
// lib/blocs/authentication/authentication_state.dart

import 'package:meta/meta.dart';
import 'package:equatable/equatable.dart';
import '../../models/models.dart';

abstract class AuthenticationState extends Equatable {
  const AuthenticationState();

  @override
  List<Object> get props => [];
}

class AuthenticationInitial extends AuthenticationState {}

class AuthenticationLoading extends AuthenticationState {}

class AuthenticationNotAuthenticated extends AuthenticationState {}

class AuthenticationAuthenticated extends AuthenticationState {
  final User user;

  AuthenticationAuthenticated({@required this.user});

  @override
  List<Object> get props => [user];
}

class AuthenticationFailure extends AuthenticationState {
  final String message;

  AuthenticationFailure({@required this.message});

  @override
  List<Object> get props => [message];
}

```

The states are self-explanatory so we are not going to discuss them further. Let us now turn our attention to the `BLoC`. Create a new file and call it `authentication_bloc.dart`. Add the following code:

```java
// lib/blocs/authentication/authentication_bloc.dart

import 'package:bloc/bloc.dart';

import 'authentication_event.dart';
import 'authentication_state.dart';
import '../../services/services.dart';

class AuthenticationBloc extends Bloc<AuthenticationEvent, AuthenticationState> {
  final AuthenticationService _authenticationService;

  AuthenticationBloc(AuthenticationService authenticationService)
      : assert(authenticationService != null),
        _authenticationService = authenticationService,
        super(AuthenticationInitial());

  @override
  Stream<AuthenticationState> mapEventToState(AuthenticationEvent event) async* {
    if (event is AppLoaded) {
      yield* _mapAppLoadedToState(event);
    }

    if (event is UserLoggedIn) {
      yield* _mapUserLoggedInToState(event);
    }

    if (event is UserLoggedOut) {
      yield* _mapUserLoggedOutToState(event);
    }
  }

  Stream<AuthenticationState> _mapAppLoadedToState(AppLoaded event) async* {
    yield AuthenticationLoading();
    try {
      await Future.delayed(Duration(milliseconds: 500)); // a simulated delay
      final currentUser = await _authenticationService.getCurrentUser();

      if (currentUser != null) {
        yield AuthenticationAuthenticated(user: currentUser);
      } else {
        yield AuthenticationNotAuthenticated();
      }
    } catch (e) {
      yield AuthenticationFailure(message: e.message ?? 'An unknown error occurred');
    }
  }

  Stream<AuthenticationState> _mapUserLoggedInToState(UserLoggedIn event) async* {
    yield AuthenticationAuthenticated(user: event.user);
  }

  Stream<AuthenticationState> _mapUserLoggedOutToState(UserLoggedOut event) async* {
    await _authenticationService.signOut();
    yield AuthenticationNotAuthenticated();
  }
}


```

We saw in the previous post that a BLoC is responsible for converting a stream of incoming events into a stream of outgoing states. This is done inside the `mapEventToState` method which takes in an `AuthenticationEvent` and returns a stream of `AuthenticationState`. This method is called every time an `AuthenticationEvent` is added to the events stream. Note that the method does not return a new stream each time an event is fired but it `yield`s the state into the existing stream. For more information on streams in dart check out [this page](https://dart.dev/tutorials/language/streams).

Inside the `mapEventToState` method we check what event type was added to the stream and we map it to a method that does business logic for that specific event. For instance, if `AppLoaded` event is received we call `_mapAppLoadedToState` which in turn calls the `AuthenticationService` to check if there is a currently logged in user. If there is, it will `yield` `AuthenticationAuthenticated` state otherwise it will `yield` `AuthenticationNotAuthenticated`.

#### Login

Now let us write code for the login `BLoC`. Like with the authentication `BLoC` above we will create a new directory inside `lib/blocs` called `login` and first create the `login_event.dart` file to which we add the following code:

```java
// lib/blocs/login/login_event.dart

import 'package:meta/meta.dart';
import 'package:equatable/equatable.dart';

abstract class LoginEvent extends Equatable {
  @override
  List<Object> get props => [];
}

class LoginInWithEmailButtonPressed extends LoginEvent {
  final String email;
  final String password;

  LoginInWithEmailButtonPressed({@required this.email, @required this.password});

  @override
  List<Object> get props => [email, password];
}

```

As you can see, we only have one login event which will be fired when a user presses the login button in the UI. Let us now add the login states inside the `login_state.dart` file:

```java
// lib/blocs/login/login_state.dart

import 'package:meta/meta.dart';
import 'package:equatable/equatable.dart';


abstract class LoginState extends Equatable{
  @override
  List<Object> get props => [];
}

class LoginInitial extends LoginState {}

class LoginLoading extends LoginState {}

class LoginSuccess extends LoginState {}

class LoginFailure extends LoginState {
  final String error;

  LoginFailure({@required this.error});

  @override
  List<Object> get props => [error];
}

```

Finally, let us go and write the login BLoC which will convert a stream of incoming login events into a stream of outgoing login states.

```java
// lib/blocs/login/login_bloc.dart

import 'package:bloc/bloc.dart';
import 'login_event.dart';
import 'login_state.dart';
import '../authentication/authentication.dart';
import '../../exceptions/exceptions.dart';
import '../../services/services.dart';

class LoginBloc extends Bloc<LoginEvent, LoginState> {
  final AuthenticationBloc _authenticationBloc;
  final AuthenticationService _authenticationService;

  LoginBloc(AuthenticationBloc authenticationBloc, AuthenticationService authenticationService)
      : assert(authenticationBloc != null),
        assert(authenticationService != null),
        _authenticationBloc = authenticationBloc,
        _authenticationService = authenticationService,
        super(LoginInitial());

  @override
  Stream<LoginState> mapEventToState(LoginEvent event) async* {
    if (event is LoginInWithEmailButtonPressed) {
      yield* _mapLoginWithEmailToState(event);
    }
  }

  Stream<LoginState> _mapLoginWithEmailToState(LoginInWithEmailButtonPressed event) async* {
    yield LoginLoading();
    try {
      final user = await _authenticationService.signInWithEmailAndPassword(event.email, event.password);
      if (user != null) {
        _authenticationBloc.add(UserLoggedIn(user: user));
        yield LoginSuccess();
        yield LoginInitial();
      } else {
        yield LoginFailure(error: 'Something very weird just happened');
      }
    } on AuthenticationException catch (e) {
      yield LoginFailure(error: e.message);
    } catch (err) {
      yield LoginFailure(error: err.message ?? 'An unknown error occured');
    }
  }
}

```

As you can see, the `LoginBloc` depends on `AuthenticationBloc`. This is because after a user has successfully logged in we want to notify the `AuthenticationBloc` that a new event (`UserLoggedIn`) has occured so the authentication bloc can push out the `AuthenticationAuthenticated` state to all listening widgets. This is not the only way to do it but I decided to do it this way.

### User Interface

Now that we have finished all the `BLoC`s needed for the app let us turn our attention to the UI. We are going to create a directory called `pages` where all the pages for our app will reside.

#### Login Page

Inside the `pages` directory create a new file called `login_page.dart` and add the following code:

```java
// lib/pages/login_page.dart

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/blocs.dart';
import '../services/services.dart';

class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Login'),
      ),
      body: SafeArea(
        minimum: const EdgeInsets.all(16),
        child: BlocBuilder<AuthenticationBloc, AuthenticationState>(
          builder: (context, state){
            final authBloc = BlocProvider.of<AuthenticationBloc>(context);
            if (state is AuthenticationNotAuthenticated){
              return _AuthForm(); // show authentication form
            }
            if (state is AuthenticationFailure) {
              // show error message
              return Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    crossAxisAlignment: CrossAxisAlignment.center,
                    children: <Widget>[
                      Text(state.message),
                      FlatButton(
                        textColor: Theme.of(context).primaryColor,
                        child: Text('Retry'),
                        onPressed: () {
                          authBloc.add(AppLoaded());
                        },
                      )
                    ],
                  ));
            }
            // show splash screen
            return Center(
              child: CircularProgressIndicator(
                strokeWidth: 2,
              ),
            );
          },
        )
      ),
    );
  }
}

class _AuthForm extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final authService = RepositoryProvider.of<AuthenticationService>(context);
    final authBloc = BlocProvider.of<AuthenticationBloc>(context);

    return Container(
      alignment: Alignment.center,
      child: BlocProvider<LoginBloc>(
        create: (context) => LoginBloc(authBloc, authService),
        child: _SignInForm(),
      ),
    );
  }
}

class _SignInForm extends StatefulWidget {
  @override
  __SignInFormState createState() => __SignInFormState();
}

class __SignInFormState extends State<_SignInForm> {

  final GlobalKey<FormState> _key = GlobalKey<FormState>();
  final _passwordController = TextEditingController();
  final _emailController = TextEditingController();
  bool _autoValidate = false;

  @override
  Widget build(BuildContext context) {
    final _loginBloc = BlocProvider.of<LoginBloc>(context);

    _onLoginButtonPressed () {
      if (_key.currentState.validate()) {
        _loginBloc.add(LoginInWithEmailButtonPressed(
            email: _emailController.text,
            password: _passwordController.text
        ));
      }else {
        setState(() {
          _autoValidate = true;
        });
      }
    }
    return BlocListener<LoginBloc, LoginState>(
      listener: (context, state){
        if (state is LoginFailure){
          _showError(state.error);
        }
      },
      child: BlocBuilder<LoginBloc, LoginState>(
        builder: (context, state){
          if (state is LoginLoading){
            return Center(
              child: CircularProgressIndicator(),
            );
          }
          return Form(
            key: _key,
            autovalidate: _autoValidate,
            child: SingleChildScrollView(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: <Widget>[
                  TextFormField(
                    decoration: InputDecoration(
                      labelText: 'Email address',
                      filled: true,
                      isDense: true,
                    ),
                    controller: _emailController,
                    keyboardType: TextInputType.emailAddress,
                    autocorrect: false,
                    validator: (value){
                      if (value == null){
                        return 'Email is required.';
                      }
                      return null;
                    },
                  ),
                  SizedBox(
                    height: 12,
                  ),
                  TextFormField(
                    decoration: InputDecoration(
                      labelText: 'Password',
                      filled: true,
                      isDense: true,
                    ),
                    obscureText: true,
                    controller: _passwordController,
                    validator: (value) {
                      if (value == null){
                        return 'Password is required.';
                      }
                      return null;
                    },
                  ),
                  const SizedBox(
                    height: 16,
                  ),
                  RaisedButton(
                    color: Theme.of(context).primaryColor,
                    textColor: Colors.white,
                    padding: const EdgeInsets.all(16),
                    shape: new RoundedRectangleBorder(borderRadius: new BorderRadius.circular(8.0)),
                    child: Text('LOG IN'),
                    onPressed: state is LoginLoading ? () {} : _onLoginButtonPressed,
                  )
                ],
              ),
            ),
          );
        },
      ),
    );
  }

  void _showError(String error) {
    Scaffold.of(context).showSnackBar(
      SnackBar(
        content: Text(error),
        backgroundColor: Theme.of(context).errorColor,
      )
    );
  }

}

```

Let us talk a little bit about the classes that are provided by the `flutter_bloc` library.

#### 1. `BlocProvider`

This is a widget that is responsible for injecting a `BLoC` instance to its children. Inside the `_AuthForm` widget we provide the `LoginBloc` to the `_SignInForm` widget and all its children. This means all the child widgets of `_SignInForm` will also have access to the bloc as it is propagated down the widget tree. To access the `BLoC` you use the `BlocProvider.of<T>(context)` method. For instance, if you want to access the `AuthenticationBloc` you call `BlocProvider.of<AuthenticationBloc>(context)`.

#### 2. `BlocBuilder`

This widget is responsible for rebuilding the UI when the state changes. For instance inside the `LoginPage` widget we have a `BlocBuilder` that listens to the `AuthenticationState` stream. Whenever the state changes the `BlocBuilder` will build a new widget: the authentication form, error message or the splash screen depending on the state.

#### 3. `BlocListener`

This widget also listens to changes in state but unlike the `BlocBuilder` it does not rebuild the UI. You can choose whatever you want to do when the state changes -- navigate to another screen, show a message, call a service etc. In our example inside the `__SignInFormState` widget the `BlocListener` listens to changes to `LoginState`. When a `LoginFailure` is received a toast notification will be displayed notifying the user of the error.

For more indepth information on these widgets check out this [website](https://bloclibrary.dev/#/flutterbloccoreconcepts).

### Home Page

Let us now create a `home_page.dart` file inside the `pages` directory and add the following code:

```java
// lib/pages/home_page.dart

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_bloc_authentication/blocs/authentication/authentication.dart';
import '../models/models.dart';

class HomePage extends StatelessWidget {
  final User user;

  const HomePage({Key key, this.user}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final authBloc = BlocProvider.of<AuthenticationBloc>(context);
    return Scaffold(
      appBar: AppBar(
        title: Text('Home Page'),
      ),
      body: SafeArea(
        minimum: const EdgeInsets.all(16),
        child: Center(
          child: Column(
            children: <Widget>[
              Text(
                'Welcome, ${user.name}',
                style: TextStyle(
                  fontSize: 24
                ),
              ),
              const SizedBox(
                height: 12,
              ),
              FlatButton(
                textColor: Theme.of(context).primaryColor,
                child: Text('Logout'),
                onPressed: (){
                  // Add UserLoggedOut to authentication event stream.
                  authBloc.add(UserLoggedOut());
                },
              )
            ],
          ),
        ),
      ),
    );
  }
}

```

This is a simple page that displays the name of the logged in user and has a logout button.

#### Putting everything together

Let us now go to `main.dart` and add the following code:

```java
// lib/main.dart

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'blocs/blocs.dart';
import 'services/services.dart';
import 'pages/pages.dart';

void main() => runApp(
        // Injects the Authentication service
        RepositoryProvider<AuthenticationService>(
          create: (context) {
            return FakeAuthenticationService();
          },
          // Injects the Authentication BLoC
          child: BlocProvider<AuthenticationBloc>(
            create: (context) {
              final authService = RepositoryProvider.of<AuthenticationService>(context);
              return AuthenticationBloc(authService)..add(AppLoaded());
            },
            child: MyApp(),
          ),
    ));

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Authentication Demo',
      theme: ThemeData(
        primarySwatch: Colors.teal,
      ),
      // BlocBuilder will listen to changes in AuthenticationState
      // and build an appropriate widget based on the state.
      home: BlocBuilder<AuthenticationBloc, AuthenticationState>(
        builder: (context, state) {
          if (state is AuthenticationAuthenticated) {
            // show home page
            return HomePage(
              user: state.user,
            );
          }
          // otherwise show login page
          return LoginPage();
        },
      ),
    );
  }
}

```

As you may have noticed there is another widget in the `flutter_bloc` library that we have used -- `RepositoryProvider`. This widget is responsible for injecting repositories (any class that is not a `BLoC`) to other widgets down the widget tree. You can also see that it is the first widget in our app followed by the `AuthenticationBloc` provider. This is because we want our entire app to have access to the `AuthenticationBloc` which in turn needs to have access to the `AuthenticationService`. The `AuthenticationService` is also a dependency of the `LoginBloc`. Repositories are accessed using `RepositoryProvider.of<T>(context)`.

We are done! You may run your app it should behave like the GIF below. Like I said earlier not all the code is included here so you will have to check out the complete project on [GitHub](https://github.com/vince-nyanga/flutter_bloc_authentication/).

<figure class="third center">
<img src="{{ site.baseurl }}/images/flutter/bloc/app.gif" alt="Flutter app">
<figcaption>App showing authentication flow built using the BLoC pattern</figcaption>
</figure>
Once again, thank you so much for taking your time to read.
