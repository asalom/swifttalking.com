---
layout: post
title:  Writing mocks manually in Swift
description: When writing unit tests we want to test the interactions between the subject under test and its dependencies. In this blog post you'll learn how.
author: asalom
tags:   [swift]
---

When writing unit tests we want to check the interactions between the subject under test or `sut` and its dependencies. In other programming languages, like Objective-C we can do this with one of the multiple mocking frameworks available such as [OCMock](http://ocmock.org/). Unfortunately in Swift we don't have any framework like these available and the reason for this is that Swift’s Reflection is limited and provides read-only access to a subset of type metadata. In other words, we can't modify class definitions in runtime. This leaves us with two options: Write mocks manually or automatically with some short of code generator. Today we'll talk about the first option.

### Writing mocks manually
_Protocols_ are very useful when writing mocks. A protocol defines a blueprint of methods, properties and other requirements that adopting classes, structs or enumerations must implement. A protocol can be adopted by one or several classes. Let's say we have the following scenario: A presenter can fetch users through a repository and update a view when the users are retrieved. We can define the view and the repository as follows:

```swift
struct User: Equatable {
  let name: String
  let lastname: String
}

protocol UserView: class {
  func update(user: User)
}

protocol UserRepository {
  typealias UserCallback = (User) -> Void
  func fetchUser(result: @escaping UserCallback)
}
```

The actual implementations are irrelevant for this demonstration. Notice how `UserView` is defined as a `class`. The reason for this is so we can later declare a property of view as `weak` so we don't create a retain cycle between the view which will be implemented by a subclass of `UIViewController` and `UserPresenter`.

```swift
class UserPresenter {
  private weak var view: UserView?
  private let repository: UserRepository

  init(view: UserView, repository: UserRepository) {
    self.view = view
    self.repository = repository
  }

  func loadUser() {
    repository.fetchUser { user in
      self.view?.update(user: user)
    }
  }
}
```

So far so good. The next step of our development cycle will be testing the interactions of `UserPresenter` with its dependencies. To do this, we'll go to our test target and create two new implementations of `UserView` and `UserRepository`

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

The mock implemtentations provide access control variables for the defined methods so later we can ask questions to those instances from the actual test.

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