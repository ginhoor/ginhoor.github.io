- # Navigator class 

  A widget that manages a set of child widgets with a stack discipline.

  Many apps have a navigator near the top of their widget hierarchy in order to display their logical history using an [Overlay](https://docs.flutter.io/flutter/widgets/Overlay-class.html) with the most recently visited pages visually on top of the older pages. Using this pattern lets the navigator visually transition from one page to another by moving the widgets around in the overlay. Similarly, the navigator can be used to show a dialog by positioning the dialog widget above the current page.

  ## Using the Navigator

  Mobile apps typically reveal their contents via full-screen elements called "screens" or "pages". In Flutter these elements are called routes and they're managed by a [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html) widget. The navigator manages a stack of [Route](https://docs.flutter.io/flutter/widgets/Route-class.html) objects and provides methods for managing the stack, like [Navigator.push](https://docs.flutter.io/flutter/widgets/Navigator/push.html) and [Navigator.pop](https://docs.flutter.io/flutter/widgets/Navigator/pop.html).

  ### Displaying a full-screen route

  Although you can create a navigator directly, it's most common to use the navigator created by a [WidgetsApp](https://docs.flutter.io/flutter/widgets/WidgetsApp-class.html) or a [MaterialApp](https://docs.flutter.io/flutter/material/MaterialApp-class.html) widget. You can refer to that navigator with [Navigator.of](https://docs.flutter.io/flutter/widgets/Navigator/of.html).

  A [MaterialApp](https://docs.flutter.io/flutter/material/MaterialApp-class.html) is the simplest way to set things up. The [MaterialApp](https://docs.flutter.io/flutter/material/MaterialApp-class.html)'s home becomes the route at the bottom of the [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)'s stack. It is what you see when the app is launched.

  ```dart
  void main() {
    runApp(MaterialApp(home: MyAppHome()));
  }
  ```

  To push a new route on the stack you can create an instance of [MaterialPageRoute](https://docs.flutter.io/flutter/material/MaterialPageRoute-class.html) with a builder function that creates whatever you want to appear on the screen. For example:

  ```dart
  Navigator.push(context, MaterialPageRoute<void>(
    builder: (BuildContext context) {
      return Scaffold(
        appBar: AppBar(title: Text('My Page')),
        body: Center(
          child: FlatButton(
            child: Text('POP'),
            onPressed: () {
              Navigator.pop(context);
            },
          ),
        ),
      );
    },
  ));
  ```

  The route defines its widget with a builder function instead of a child widget because it will be built and rebuilt in different contexts depending on when it's pushed and popped.

  As you can see, the new route can be popped, revealing the app's home page, with the Navigator's pop method:

  ```dart
  Navigator.pop(context);
  ```

  It usually isn't necessary to provide a widget that pops the Navigator in a route with a [Scaffold](https://docs.flutter.io/flutter/material/Scaffold-class.html) because the Scaffold automatically adds a 'back' button to its AppBar. Pressing the back button causes [Navigator.pop](https://docs.flutter.io/flutter/widgets/Navigator/pop.html)to be called. On Android, pressing the system back button does the same thing.

  ### Using named navigator routes

  Mobile apps often manage a large number of routes and it's often easiest to refer to them by name. Route names, by convention, use a path-like structure (for example, '/a/b/c'). The app's home page route is named '/' by default.

  The [MaterialApp](https://docs.flutter.io/flutter/material/MaterialApp-class.html) can be created with a `Map<String, WidgetBuilder>` which maps from a route's name to a builder function that will create it. The [MaterialApp](https://docs.flutter.io/flutter/material/MaterialApp-class.html) uses this map to create a value for its navigator's [onGenerateRoute](https://docs.flutter.io/flutter/widgets/Navigator/onGenerateRoute.html) callback.

  ```dart
  void main() {
    runApp(MaterialApp(
      home: MyAppHome(), // becomes the route named '/'
      routes: <String, WidgetBuilder> {
        '/a': (BuildContext context) => MyPage(title: 'page A'),
        '/b': (BuildContext context) => MyPage(title: 'page B'),
        '/c': (BuildContext context) => MyPage(title: 'page C'),
      },
    ));
  }
  ```

  To show a route by name:

  ```dart
  Navigator.pushNamed(context, '/b');
  ```

  ### Routes can return a value

  When a route is pushed to ask the user for a value, the value can be returned via the [pop](https://docs.flutter.io/flutter/widgets/Navigator/pop.html) method's result parameter.

  Methods that push a route return a [Future](https://docs.flutter.io/flutter/dart-async/Future-class.html). The Future resolves when the route is popped and the [Future](https://docs.flutter.io/flutter/dart-async/Future-class.html)'s value is the [pop](https://docs.flutter.io/flutter/widgets/Navigator/pop.html) method's `result` parameter.

  For example if we wanted to ask the user to press 'OK' to confirm an operation we could `await` the result of [Navigator.push](https://docs.flutter.io/flutter/widgets/Navigator/push.html):

  ```dart
  bool value = await Navigator.push(context, MaterialPageRoute<bool>(
    builder: (BuildContext context) {
      return Center(
        child: GestureDetector(
          child: Text('OK'),
          onTap: () { Navigator.pop(context, true); }
        ),
      );
    }
  ));
  ```

  If the user presses 'OK' then value will be true. If the user backs out of the route, for example by pressing the Scaffold's back button, the value will be null.

  When a route is used to return a value, the route's type parameter must match the type of [pop](https://docs.flutter.io/flutter/widgets/Navigator/pop.html)'s result. That's why we've used `MaterialPageRoute<bool>` instead of `MaterialPageRoute<void>` or just`MaterialPageRoute`. (If you prefer to not specify the types, though, that's fine too.)

  ### Popup routes

  Routes don't have to obscure the entire screen. [PopupRoute](https://docs.flutter.io/flutter/widgets/PopupRoute-class.html)s cover the screen with a [ModalRoute.barrierColor](https://docs.flutter.io/flutter/widgets/ModalRoute/barrierColor.html) that can be only partially opaque to allow the current screen to show through. Popup routes are "modal" because they block input to the widgets below.

  There are functions which create and show popup routes. For example: [showDialog](https://docs.flutter.io/flutter/material/showDialog.html), [showMenu](https://docs.flutter.io/flutter/material/showMenu.html), and [showModalBottomSheet](https://docs.flutter.io/flutter/material/showModalBottomSheet.html). These functions return their pushed route's Future as described above. Callers can await the returned value to take an action when the route is popped, or to discover the route's value.

  There are also widgets which create popup routes, like [PopupMenuButton](https://docs.flutter.io/flutter/material/PopupMenuButton-class.html) and [DropdownButton](https://docs.flutter.io/flutter/material/DropdownButton-class.html). These widgets create internal subclasses of PopupRoute and use the Navigator's push and pop methods to show and dismiss them.

  ### Custom routes

  You can create your own subclass of one of the widget library route classes like [PopupRoute](https://docs.flutter.io/flutter/widgets/PopupRoute-class.html), [ModalRoute](https://docs.flutter.io/flutter/widgets/ModalRoute-class.html), or [PageRoute](https://docs.flutter.io/flutter/widgets/PageRoute-class.html), to control the animated transition employed to show the route, the color and behavior of the route's modal barrier, and other aspects of the route.

  The [PageRouteBuilder](https://docs.flutter.io/flutter/widgets/PageRouteBuilder-class.html) class makes it possible to define a custom route in terms of callbacks. Here's an example that rotates and fades its child when the route appears or disappears. This route does not obscure the entire screen because it specifies `opaque: false`, just as a popup route does.

  ```dart
  Navigator.push(context, PageRouteBuilder(
    opaque: false,
    pageBuilder: (BuildContext context, _, __) {
      return Center(child: Text('My PageRoute'));
    },
    transitionsBuilder: (___, Animation<double> animation, ____, Widget child) {
      return FadeTransition(
        opacity: animation,
        child: RotationTransition(
          turns: Tween<double>(begin: 0.5, end: 1.0).animate(animation),
          child: child,
        ),
      );
    }
  ));
  ```

  The page route is built in two parts, the "page" and the "transitions". The page becomes a descendant of the child passed to the `buildTransitions` method. Typically the page is only built once, because it doesn't depend on its animation parameters (elided with `_` and `__` in this example). The transition is built on every frame for its duration.

  ### Nesting Navigators

  An app can use more than one Navigator. Nesting one Navigator below another Navigator can be used to create an "inner journey" such as tabbed navigation, user registration, store checkout, or other independent journeys that represent a subsection of your overall application.

  #### Real World Example

  It is standard practice for iOS apps to use tabbed navigation where each tab maintains its own navigation history. Therefore, each tab has its own [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html), creating a kind of "parallel navigation."

  In addition to the parallel navigation of the tabs, it is still possible to launch full-screen pages that completely cover the tabs. For example: an on-boarding flow, or an alert dialog. Therefore, there must exist a "root" [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html) that sits above the tab navigation. As a result, each of the tab's [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s are actually nested [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s sitting below a single root [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html).

  The nested [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s for tabbed navigation sit in `WidgetApp` and [CupertinoTabView](https://docs.flutter.io/flutter/cupertino/CupertinoTabView-class.html), so you don't need to worry about nested [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s in this situation, but it's a real world example where nested [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s are used.

  #### Sample Code

  The following example demonstrates how a nested [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html) can be used to present a standalone user registration journey.

  Even though this example uses two [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s to demonstrate nested [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s, a similar result is possible using only a single [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html).

  ```dart
  class MyApp extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return MaterialApp(
        // ...some parameters omitted...
        // MaterialApp contains our top-level Navigator
        initialRoute: '/',
        routes: {
          '/': (BuildContext context) => HomePage(),
          '/signup': (BuildContext context) => SignUpPage(),
        },
      );
    }
  }
  
  class SignUpPage extends StatelessWidget {
   @override
   Widget build(BuildContext context) {
     // SignUpPage builds its own Navigator which ends up being a nested
     // Navigator in our app.
     return Navigator(
       initialRoute: 'signup/personal_info',
       onGenerateRoute: (RouteSettings settings) {
         WidgetBuilder builder;
         switch (settings.name) {
           case 'signup/personal_info':
             // Assume CollectPersonalInfoPage collects personal info and then
             // navigates to 'signup/choose_credentials'.
             builder = (BuildContext _) => CollectPersonalInfoPage();
             break;
           case 'signup/choose_credentials':
             // Assume ChooseCredentialsPage collects new credentials and then
             // invokes 'onSignupComplete()'.
             builder = (BuildContext _) => ChooseCredentialsPage(
               onSignupComplete: () {
                 // Referencing Navigator.of(context) from here refers to the
                 // top level Navigator because SignUpPage is above the
                 // nested Navigator that it created. Therefore, this pop()
                 // will pop the entire "sign up" journey and return to the
                 // "/" route, AKA HomePage.
                 Navigator.of(context).pop();
               },
             );
             break;
           default:
             throw Exception('Invalid route: ${settings.name}');
         }
         return MaterialPageRoute(builder: builder, settings: settings);
       },
     );
   }
  }
  ```

  [Navigator.of](https://docs.flutter.io/flutter/widgets/Navigator/of.html) operates on the nearest ancestor [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html) from the given [BuildContext](https://docs.flutter.io/flutter/widgets/BuildContext-class.html). Be sure to provide a [BuildContext](https://docs.flutter.io/flutter/widgets/BuildContext-class.html) below the intended [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html), especially in large `build` methods where nested [Navigator](https://docs.flutter.io/flutter/widgets/Navigator-class.html)s are created. The [Builder](https://docs.flutter.io/flutter/widgets/Builder-class.html) widget can be used to access a [BuildContext](https://docs.flutter.io/flutter/widgets/BuildContext-class.html) at a desired location in the widget subtree.

  - Inheritance

    [Object](https://docs.flutter.io/flutter/dart-core/Object-class.html)> [Diagnosticable](https://docs.flutter.io/flutter/foundation/Diagnosticable-class.html)> [DiagnosticableTree](https://docs.flutter.io/flutter/foundation/DiagnosticableTree-class.html)> [Widget](https://docs.flutter.io/flutter/widgets/Widget-class.html)> [StatefulWidget](https://docs.flutter.io/flutter/widgets/StatefulWidget-class.html)> Navigator