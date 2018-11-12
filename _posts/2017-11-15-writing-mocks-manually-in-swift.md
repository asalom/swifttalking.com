---
layout: post
title:  Writing mocks manually in Swift
description: When writing unit tests we want to test the interactions between the subject under test and its dependencies. In this blog post you'll learn how.
author: asalom
tags:   [swift]
---

When writing unit tests we want to check the interactions between the subject under test or `sut` and its dependencies. In other programming languages, like Objective-C we can do this with one of the multiple mocking frameworks available such as [OCMock](http://ocmock.org/). Unfortunately in Swift we don't have any framework like these available and the reason for this is that Swift’s reflection capabilities are limited and provides read-only access to a subset of type metadata. In other words, we can't modify class definitions in runtime. This leaves us with two options: Write mocks manually or automatically with some short of code generator. Today we'll talk about the first option.

### Writing mocks manually
_Protocols_ are very useful when writing mocks. A protocol defines a blueprint of methods, properties and other requirements that adopting classes, structs or enumerations must implement. A protocol can be adopted by one or several classes. Let's say we have the following scenario: A presenter can fetch a user through a repository and update a view when the user is retrieved. We can define the view and the repository as follows:

```swift
struct User: Equatable {
  let name: String
  let lastname: String
}

protocol UserView {
  func update(user: User)
}

protocol UserRepository {
  typealias UserCallback = (User) -> Void
  func fetchUser(result: @escaping UserCallback)
}
```

The actual implementations for the repository and the view are irrelevant for this demonstration. Let's now create our presenter which will fetch the user's data and update the view.

```swift
class UserPresenter {
  private let view: UserView
  private let repository: UserRepository

  init(view: UserView, repository: UserRepository) {
    self.view = view
    self.repository = repository
  }

  func loadUser() {
    repository.fetchUser { user in
      self.view.update(user: user)
    }
  }
}
```

So far so good. The next step will be testing the interactions of `UserPresenter` with its dependencies. To do this, we'll go to our test target and create two new implementations of `UserView` and `UserRepository`.

```swift
class UserViewMock: UserView {
  var updateParam: User?
  func update(user: User) {
    updateParam = user
  }
}

class UserRepositoryMock: UserRepository {
  var fetchUserParam: UserCallback?
  var fetchUserWasCalled = false
  func fetchUser(result: @escaping UserCallback) {
    fetchUserParam = result
    fetchUserWasCalled = true
  }
}
```

We added control variables to the mocks so we can later check from the unit tests that certain functions were called and what kind of parameters were passed to them.

We can now create instances of `UserViewMock` and `UserRepositoryMock`, inject them into `UserPresenter` and validate that the interactions work as expected. We can even fake a response from the fetching function and make it return any data we want.

```swift
class UserPresenterTests: XCTestCase {
  var view: UserViewMock!
  var repository: UserRepositoryMock!
  var sut: UserPresenter!

  override func setUp() {
    super.setUp()

    view = UserViewMock()
    repository = UserRepositoryMock()
    sut = UserPresenter(view: view, repository: repository)
  }

  func testLoadUser() {
    sut.loadUser()

    XCTAssertTrue(repository.fetchUserWasCalled)

    let expecedUser = User(name: "Alex", lastname: "Salom")
    repository.fetchUserParam?(expecedUser)

    XCTAssertEqual(view.updateParam, expecedUser)
  }
}
```

### But... what about unowned code?

We saw that writing mocks for code that we own is very easy if we make our classes implement certain protocols but what happens with code which we do not own and we can't modify? 

The answer is again we need to use _protocols_. We need to extend the code that we do not own with our own protocols.
