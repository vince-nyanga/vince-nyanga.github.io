---
title: "Building An Authentication Flow In Flutter Using The GetX Library"
date: 2021-01-08
tags: [Flutter, Mobile]
---

A little under a year ago I wrote an [article]({{ site.baseurl}}/flutter-authentication-with-bloc-architecture) where I showed how to create an authentication flow using BLoC. On New Year's Day (2021) as I was catching up with everything Flutter after being off the platform for a little over 6 months, I came across an awesome library called [GetX](https://pub.dev/packages/get/). After reading about it for a couple of hours I decided to learn it by creating something. In this post I will create an authentication flow using `GetX`.

## What Is GetX

According to the [docs](https://pub.dev/packages/get/#about-get) `GetX` is an ultra-light library for Flutter that combines high performance state management, intelligent dependency injection and route management. It has 3 basic principles at its core -- performance, productivity and organization. If you want to learn more about this library check out the docs link above.

In this post I will make use of `GetX`'s state management and dependency injection to create an authentication flow similar to the one I did using [Bloc]({{ site.baseurl}}/flutter-authentication-with-bloc-architecture). I am not going to show every line of code for brevity. All the source code is available on [GitHub](https://github.com/vince-nyanga/flutter_getx_authentication). Let's get started.

## The App

Create a new flutter app and add the following libraries:

```
get: ^3.24.0
equatable: ^1.0.2
```

Once the packages have been installed create a directory called `features/` where we are going to add all our features for the project. We will put everything that relates to a feature (or module) inside its own feature folder. This helps with organization in my opinion. We will start off with the authentication feature.

### Authentication Feature

Add a directory `authentication/` inside the `features` directory and add `authentication_state.dart` therein. Inside this file add the following code:

```java
// features/authentication/authentication_state.dart


class AuthenticationState extends Equatable {
  const AuthenticationState();

  @override
  List<Object> get props => [];
}

class AuthenticationLoading extends AuthenticationState {}

class UnAuthenticated extends AuthenticationState {}

class Authenticated extends AuthenticationState {
  final User user;

  Authenticated({@required this.user});

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

These are the authentication states for our app. We are going to make use of them later when we build the UI.

Now let's create our authentication controller. Add `authentication_controller.dart` to the `authentication` folder and add the following code:

```java
// features/authentication/authentication_controller.dart


class AuthenticationController extends GetxController {
  final AuthenticationService _authenticationService;
  final _authenticationStateStream = AuthenticationState().obs;

  AuthenticationState get state => _authenticationStateStream.value;

  AuthenticationController(this._authenticationService);

  // Called immediately after the contoller is allocated in memory.
  @override
  void onInit() {
    _getAuthenticatedUser();
    super.onInit();
  }

  Future<void> signIn(String email, String password) async {
    final user = await _authenticationService.signInWithEmailAndPassword(email, password);
    _authenticationStateStream.value = Authenticated(user: user);
  }

  void signOut() async {
    await _authenticationService.signOut();
    _authenticationStateStream.value = UnAuthenticated();
  }

  void _getAuthenticatedUser() async {
    _authenticationStateStream.value = AuthenticationLoading();

    final user = await _authenticationService.getCurrentUser();

    if (user == null) {
      _authenticationStateStream.value = UnAuthenticated();
    } else {
      _authenticationStateStream.value = Authenticated(user: user);
    }
  }
}
```

The `AuthenticationController` extends `GetX`'s `GetxController` class. This class is lifecycle aware and will get disposed once there is nothing depending on it. This class, as suggested in the name, is a controller that contains logic that will be used by our view in a classic MVC architecture.

As you may have noticed GetX has an extension function `.obs` that turns any object `T` into a stream -- `Rx<T>`. This is what we have done with our `_authenticationStateStream` property. I am a huge fan of reactive programming so we are going to be using it in this app.

### Login Feature

We are now going to implement our login feature. I'm not going to talk about login states, get the source code on [GitHub](https://github.com/vince-nyanga/flutter_getx_authentication/tree/main/src/lib/features/authentication/login) if you want to look at it. Let's add our login controller:

```java
// features/authentication/login/login_controller.dart

class LoginController  extends GetxController {
  final AuthenticationController _authenticationController = Get.find();

  final _loginStateStream = LoginState().obs;

  LoginState get state => _loginStateStream.value;

  void login(String email, String password) async {
    _loginStateStream.value = LoginLoading();

    try{
      await _authenticationController.signIn(email, password);
      _loginStateStream.value = LoginState();
    } on AuthenticationException catch(e){
      _loginStateStream.value = LoginFailure(error: e.message);
    }
  }
}
```

In this controller we are getting the `AuthenticationController` using `GetX`'s dependency injection framework:

```java
final AuthenticationController _authenticationController = Get.find();
```

When we call `Get.find<T>()` we will get the instance of the `AuthenticationController` that's in memory. Very cool.

Let's turn our attention to the UI and create the login screen:

```java
// features/authentication/login/login_page.dart

class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: Text('Login'),
        ),
        body: SafeArea(
          minimum: const EdgeInsets.all(16),
          child: _SignInForm(),
        ));
  }
}


class _SignInForm extends StatefulWidget {
  @override
  __SignInFormState createState() => __SignInFormState();
}

class __SignInFormState extends State<_SignInForm> {
  final _controller = Get.put(LoginController()); // inject controller

  final GlobalKey<FormState> _key = GlobalKey<FormState>();
  final _passwordController = TextEditingController();
  final _emailController = TextEditingController();
  bool _autoValidate = false;

  @override
  Widget build(BuildContext context) {
    return Obx((){
      return Form(
        key: _key,
        autovalidateMode: _autoValidate ? AutovalidateMode.always : AutovalidateMode.disabled,
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
                validator: (value) {
                  if (value == null) {
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
                  if (value == null) {
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
                onPressed: _controller.state is LoginLoading ? () {} : _onLoginButtonPressed,
              ),
              const SizedBox(height: 20,),
              if (_controller.state is LoginFailure)
                Text((_controller.state as LoginFailure).error,
                  textAlign: TextAlign.center,
                  style: TextStyle(
                      color: Get.theme.errorColor // easy way to access theme
                  ),
                ),
              if (_controller.state is LoginLoading)
                Center(child: CircularProgressIndicator(),)
            ],
          ),
        ),
      );
    });
  }

  _onLoginButtonPressed() {
    if (_key.currentState.validate()) {
      _controller.login(_emailController.text, _passwordController.text);
    } else {
      setState(() {
        _autoValidate = true;
      });
    }
  }
}

```

We are using `Obx`, a simple reactive widget by `GetX` to listen to the changes in the login state which is an Rx stream. This will enable us to render conditional widgets such as the progress indicator and the error text. I used the `Obx` in this case because it is the simplest reactive widget there-is. For more `GetX` widgets check out their documentation.

### Putting Everything Together

Let's turn our attention to the `main.dart` file and add the following code:

```java
void main() {
  initialize();
  runApp(MyApp());
}

void initialize() {
    // inject authentication controller
  Get.lazyPut(() => AuthenticationController(Get.put(FakeAuthenticationService())),);
}

class MyApp extends GetWidget<AuthenticationController> {

  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      title: 'Fluter GetX Auth',
      theme: ThemeData(
        primarySwatch: Colors.purple,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      debugShowCheckedModeBanner: false,
      home: Obx(() {
        if (controller.state is UnAuthenticated) {
          return LoginPage();
        }

        if (controller.state is Authenticated) {
          return HomePage(
            user: (controller.state as Authenticated).user,
          );
        }

        return SplashScreen();
      }),
    );
  }
}
```

As you can see `MyApp` extends `GetWidget<AuthenticationController>`. This widget gives us access to our `AuthenticationController` without us having to call `Get.find()`. The other thing to note is that instead of `MaterialApp` we now have `GetMaterialApp` widget which enables us to do cool things with `GetX`.

Like in the login page, we are using `Obx` for our `home` property so we can listen to changes on the authentication state and display the respective widgets.

That's it. A simple authentication flow using `GetX`. The complete source code is available on [GitHub](https://github.com/vince-nyanga/flutter_getx_authentication) if you want to check it out. Here is how it looks:

<figure class="third center">
<img src="{{ site.baseurl }}/images/flutter/getx/getx.gif" alt="Authentication flow with GetX">
<figcaption>Authentication flow with GetX</figcaption>
</figure>

Thanks so much for taking time to read. I hope you have learned something :blush: .

## Further Reading

- [GetX library](https://pub.dev/packages/get/#about-get)
- [Building an authentication flow in Flutter using the BLoC]({{ site.baseurl}}/flutter-authentication-with-bloc-architecture)
