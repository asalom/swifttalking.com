---
layout: post
title:  Localize strings more beautifully with an extension
author: asalom
tags:   [swift]
---

If you've worked in an App that has multiple languages you'd probably have learn to ~~hate~~ love Apple's localization API `NSLocalizedString`. We just want a localize some strings but instead we are forced to provide a comment which in most cases we'll just pass in `nil`.

> Checkout the documentation [here](https://developer.apple.com/documentation/foundation/nslocalizedstring).

I've been using small `String` extension to reduce the verbosity while keeping the API flexible for different scenarios. Let's go straight to the point and show some [snippets](https://gist.github.com/asalom/cfade69708bf300c479bea614f39bc41) with the implementation and usage.

### Implementation:

```swift
public extension String {
  public func localize(tableName tableName: String? = nil, bundle: NSBundle = NSBundle.mainBundle(), comment: String = "") -> String {
    return NSLocalizedString(
      self,
      tableName: tableName,
      bundle: bundle,
      value: "",
      comment: comment)
  }
}
```

### Usage:

Basic:

``` swift
"MENU".localize()
"MENU".localize(comment: "Menu title")
```

Or if you have other `Localizable.strings` files:

``` swift
"MENU".localize(tableName: "SomeOtherLocalizable")
```

Or if you are in a different target:

``` swift
"MENU".localize(tableName: "SomeOtherLocalizableAndTarget", bundle: NSBundle(forClass: DummyClass.self))
...
private class DummyClass { }
```

In the last example I find it useful to just create another extension inside of the target were the `Localizable.strings` file is located.

```swift
extension String {
  func localizeTargetX(comment: String = "") -> String {
    return localize(tableName: "TargetXLocalizable", bundle: Bundle(for: DummyClass.self), comment: comment)
  }
}

private class DummyClass { }
```