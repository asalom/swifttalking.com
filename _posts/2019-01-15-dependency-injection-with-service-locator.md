---
layout: post
title:  Dependency Injection with Service Locator
description: With the Service Locator pattern we can make a registry of dependencies for a given object. There is a very simple way to create one with minimal code, learn how in this post.
tags: [swift]
---

When I worked at <a href="http://welt.de">WeltN24</a>'s iOS team we liked to test. We liked it so much that we built our code to be ~~almost~~ 100% testable. We were able to achieve this by injecting all dependencies through each class' initializer. One technic to do this is to make use of Swift's default values.
><a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html">The Swift Programming Language</a>
>
> You can define a default value for any parameter in a function by assigning a value to the parameter after that parameter’s type. If a default value is defined, you can omit that parameter when calling the function.

```swift
func hello(name: String = "Alex Salom") -> String {
  return "Hello \(name)"
}

hello() // Hello Alex Salom
hello(name: "Nick Cave") // Hello Nick Cave
```

Let's say that we have an object called ```PeopleManager``` who's purpose is to, well, manage people. Let's imagine that it can fetch people from the network and save people in a database. Since we like to follow the <a href="https://en.wikipedia.org/wiki/Single_responsibility_principle">Single Responsibility Principle</a>, we want these two functionalities to be encapsulated in different objects, ```Fetcher``` and ```Database```.

```swift
protocol Database {
  func save(person: Person)
}
final class DatabaseImpl: Database { ... }

protocol Fetcher {
  func fetch() -> Person
}
final class FetcherImpl: Fetcher { ... }

```

Using Swift's default values we could inject those dependencies into ```PeopleManager``` like this:

```swift
final class PeopleManager {
  private let database: Database
  private let fetcher: Fetcher

  init(database: Database = DatabaseImpl(), fetcher: Fetcher = FetcherImpl()) {
    self.database = database
    self.fetcher = fetcher
  }

  func person() -> Person {
    return fetcher.fetch()
  }

  func addPerson() {
    let person = Person(name: "Alex Salom")
    database.save(person: person)
  }
}
```

This is very nice as it allows us to initialize ```PeopleManager``` through its empty initializer from the production code ```PeopleManager()``` but giving us the possibility to inject mocked versions of the dependencies from the tests ```PeopleManager(database: DatabaseMock(), fetcher: FetcherMock())```.

However we did find one issue with this technic. When an object has quite a few dependencies, the initializer can get out of hands having long signatures. That's why we came up with something called ```ServiceLocator```. We can think of a ```ServiceLocator``` as a registry of dependencies for a given object. The idea is that each dependency will declare its own locator so other objects can find a way to initialize those dependencies. Let's see an example with our ```Database```.

```swift
protocol Database { ... }
final class DatabaseImpl: Database { ... }

protocol DatabaseLocator {
  func database() -> Database
}

extension DatabaseLocator {
  func database() -> Database {
    return DatabaseImpl()
  }
}
```

We declared a protocol ```DatabaseLocator``` with a function that will provide us with an instance of ```Database```. We also declared a protocol extension with a default implementation of that function. Now imagine we did the same for ```Fetcher``` and we now have a ```FetcherLocator``` as well as a ```DatabaseLocator```. With those in place let's revisit ```PeopleManger```'s dependency injection.

```swift
final class PeopleManager {
  typealias ServiceLocator = DatabaseLocator & FetcherLocator
  final class ServiceLocatorImpl: ServiceLocator {}

  private let database: Database
  private let fetcher: Fetcher

  init(serviceLocator: ServiceLocator = ServiceLocatorImpl()) {
    self.database = serviceLocator.database()
    self.fetcher = serviceLocator.fetcher()
  }

  func person() -> Person {
    return fetcher.fetch()
  }

  func addPerson() {
    let person = Person(name: "Nick Cave")
    database.save(person: person)
  }
}
```

We start by declaring a ```typealias ServiceLocator``` with all the dependencies of this class. This is very convenient because only by looking at this line we see all the objects ```PeopleManager``` depends on. We then inject the ```ServiceLocator``` into the initializer ```init(serviceLocator: ServiceLocator = ServiceLocatorImpl())``` using a default value of ```ServiceLocatorImpl```.
```ServiceLocatorImpl``` doesn't need to provide any implementation because each one of the Locators have provided an extension to every method they declare. We just found a way to inject as many dependencies as we want by declaring only one variable at ```PeopleManager```'s initializer. Since all the dependencies use protocols and the locators are protocols themselves we could now build our own version of ServiceLocator that returns mocked objects and inject those from the tests.
