---
layout: post
title:  Swap out your App Delegate for testing
description: Running unit tests often is important, that's why we want our test suite to run as fast as possible. You can speed up the execution of  your unit tests by using a fake App Delegate instead of the real one.
author: asalom
tags:   [swift]
---

The best thing about unit testing is the ability to know if our production code works as expected without having to tap around in the Simulator. It is crucial that we receive this feedback fast so we can run the test suite as much as possible. That is even more important when we are practicing **TDD**.

While running our tests, Xcode firstly launches the app in the simulator like if we were normally running it and thus having the site effect of executing any code we may have in our App Delegate and initial View Controller.

### Faster, please
One thing that can slow our tests down is if our App performs expensive tasks on startup and it is not unusual to place code that synchronizes with a server in our implementation of <a href="https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/">UIAppDelegate</a>.

Luckily for us there is a way to change the production App Delegate with a *fake* that does nothing.

### Changing the App Delegate
First head to your **AppDelegate.swift** file. You'll notice that there is an attribute `@UIApplicationMain` which you'll need to **delete**.

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  // Many expensive tasks
  func hello() { }
}
```

><a href="https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Attributes.html">NSApplicationMain</a>
>
> Apply this attribute to a class to indicate that it is the application delegate. Using this attribute is equivalent to calling the NSApplicationMain(::) function and passing this class’s name as the name of the delegate class.
> If you do not use this attribute, supply a main.swift file with a main() function that calls the NSApplicationMain(::) function. For example, if your app uses a custom subclass of NSApplication as its principal class, call the NSApplicationMain function instead of using this attribute.

Next step is to add a new file and name it `main.swift`. This will give us a way to inject our *fake*.


```swift
class FakeAppDelegate: UIResponder, UIApplicationDelegate {
  var window: UIWindow?

  func application(application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [NSObject : AnyObject]?) -> Bool {
    self.window?.rootViewController = UIViewController()
    return true
  }
}

let isRunningTests = NSClassFromString("XCTestCase") != nil

if isRunningTests {
  UIApplicationMain(Process.argc, Process.unsafeArgv, nil,
    NSStringFromClass(FakeAppDelegate))
} else {
  UIApplicationMain(Process.argc, Process.unsafeArgv, nil,
    NSStringFromClass(AppDelegate))
}
```

You may notice that not only we did inject a fake *App Delegate* but also have overrode ```application: didFinishLaunchingWithOptions``` setting an empty <a href="https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/">UIViewController</a> so we also avoid any expensive initialization tasks our real root View Controller may perform.

That's all there is to it.