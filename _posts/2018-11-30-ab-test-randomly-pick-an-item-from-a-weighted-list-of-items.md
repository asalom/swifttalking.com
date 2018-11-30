---
layout: post
title: A/B Test&#58 Randomly pick an item from a weighted list of items
description: This function might come in handy if you are ever tasked with building some short of A/B testing functionality. It will help you to randomly pick an item from a weighted list of items.
author: asalom
tags:   [swift]
---

Sometimes we are considering two approaches to solve one problem and we are unsure which one to choose. We could run an experiment using prototypes and test users but sometimes we want to do this in a production App and with real users so we can later mesure the impact of one option over the others.

Imagine we have an app with three different onboardings. All onboardings are visually different but they all lead to a screen where the user is prompted to buy a subscription. We would like to know which one of those onboardings will lead to more purchases so we can discard the others.

Instead of user testing prototypes, we could know which onboarding leads to more purchases by using A/B test with different onboardings. A/B testing is a way to compare two or more versions of a feature, typically by measuring a user's response to feature A against feature B, or C, etc. and determining which of the of them is more effective. In other words, we could design 3 different onboarding experiences and show each one of them randomly to different users. We can choose to randomize the selection equaly so each variant has the same chances of showing up or we could use a weighted list where one onboarding will have more chances to appear than the others.

For all this to have any value we would need to have some short of tracking in place that later we can refer to and see how each variant performed. We won't cover in this post the tracking part as this is always dependent on other factors, such as product or marketing tooling decicions. However the idea is to show one variant, measure its performance and then track the result. 

In our example we want to show different onboardings to different users, measure how many users buy a subscription after a given onboarding vs how many users were presented this onboarding and finally track this results somewhere.

Sometime ago we had to implement exactly this, where we had to track the performance of different onboardings and later choose the best one. We had three options and we weren't sure if one of the them would be too risky for our users and since this could potentially affect our sales, we decided to introduce the risky onboarding to a limited amount of users. For this we implemented the following function.

```swift
class ABTester<T> {
  typealias WeightedItem = (item: () -> T, weight: Int)

  /** Randomly chooses a weighted item from a list of items
   - item: a weighted item
   - more: more weighted items as a variadic parameter
   - randomNumber: random number generator. This is injected so we can write tests in the function.
   - returns: the winner weigthed item
   - discussion: We force the first item to be provided so we ensure that we don't end up with an empty list of items
   */
  func ABTest(_ item: WeightedItem, _ more: WeightedItem ..., randomNumber: RandomNumber = RandomNumberImpl()) -> T {
    return [[item], more].flatMap { $0 }.sorted { (lhs: WeightedItem, rhs: WeightedItem) in
      randomNumber.generateBetween(min: 1, max: lhs.weight) >
      randomNumber.generateBetween(min: 1, max: rhs.weight)
    }.first!.item()
  }
}
```

Reading through the inline comments in the code snippet above will help you understand what each parameter represents. A `WeightedItem` takes an `item` closure which represents a variant and a `weight` which represents its chances to win. The heigher the weight with respect to the other items, the bigger the chances it will be chosen. The reason a `WightedItem` takes a closure as an `item` is so we only initialize the winner since we might have items which initialization is expensive. We could hide the closure from the users of this class by applying an [`@autoclosure`](https://www.hackingwithswift.com/example-code/language/what-is-the-autoclosure-attribute) annotation to the item closure but unfortunately we can't use them yet on [tuples](https://bugs.swift.org/browse/SR-2567) or [typealiases](https://bugs.swift.org/browse/SR-2688).

Finally we inject a `RandomNumber` object in order to allow unit testing this function. `RandomNumber`'s implementation is pretty trivial.

```swift
protocol RandomNumber {
  func generateBetween(min minValue: Int, max maxValue: Int) -> Int
}

class RandomNumberImpl: RandomNumber {
  func generateBetween(min minValue: Int, max maxValue: Int) -> Int {
    return minValue + Int(arc4random_uniform(UInt32(maxValue - minValue + 1)))
  }
}
```

With all this in place, how to use it? To continue with our onboarding example, we need to initialize an `ABTester` instance of generic type `Onboarding` and call the `ABTest` function with a weighted list of onboardings adjusting their weight depending which onboarding we want to show more often over the others.

```swift
let abTester = ABTester<Onboarding>()
let onboarding = abTester.ABTest(
  (item: { OnboardingA() }, weight: 45),
  (item: { OnboardingB() }, weight: 45),
  (item: { OnboardingC() }, weight: 10)
)

view.show(onboarding)
```

In this example there will be a 45% change to show either A or B variants while only a 10% chance to show the C variant which can be a little bit risky for us.