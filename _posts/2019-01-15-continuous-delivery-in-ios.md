---
layout: post
title: Continuous delivery in iOS
description: Building, testing and deploying our Apps is important because we'll be able to detect problems faster. I'll show you the flow that I've been using for a while.
author: asalom
tags:   [automation]
---

Keeping an App deployable at any time is very important for a number of reasons. Deploying the app very often somewhere ([Fabric](https://fabric.io/), [TestFlight](https://developer.apple.com/testflight/), etc.) means that a version will be available for anyone to test at any time and if this version is always the last version it will reduce the friction between internal consumers of the App (PO, QA, CEO, etc) and developers. I've worked for some teams that didn't have a continuous delivery system in place and most weeks one of the developers had to manually install the latest version on someone's device. This meant that a lot of hours lost into something that could be done automatically. If we ensure that the delivery method ([Fabric](https://fabric.io/), [TestFlight](https://developer.apple.com/testflight/), etc.) always contains the latest version, developers will have more time for what the do best: implement new features ~~and adding new bugs~~.

### The continuous delivery flow
In my team we work in a [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)-flow, we solve every ticket in a separate branch and when we think it's good enough we merge it back into master. Since we never commit anything directly to master, we can consider that every change in master means that we finished a task and when this happens, the continuous delivery process starts.

1. We merge into master.
2. GitHub notifies the CI: [Travis CI](https://travis-ci.com/) in our case.
3. Travis executes some stuff and calls one of our [fastlane](https://fastlane.tools/) lanes
4. [fastlane](https://fastlane.tools/) tests the app, builds a new version and deploys it to  ([Fabric](https://fabric.io/).
5. Slack and JIRA are updated with the version number.

Let's see a diagram of the flow when master gets modified.

<img width="600" alt="Flow" src="../images/posts{{page.url}}flow.png">

Let's first take a look at a simplified version of our `.travis.yml` configuration file:

```yml
os: osx
osx_image: xcode10.1
branches:
  only:
    - master
script:
  - bundle exec fastlane travis
```

Travis CI will only process a job if master is modified, or if we open a PR pointing to master. When this happens we'll execute `bundle exec fastlane travis`. Let's now see what happens inside fastlane.

```ruby
lane :travis do
  if ENV['TRAVIS_PULL_REQUEST'] != 'false'
    pull_request
  elsif current_branch =~ /^master$/
    master # In our example, we'll execute this lane
  end
end
```

In this simplified example, our fastlane implementation can perform two flows, one when we open a Pull Request and the second one which we'll see in more detail, when master gets modified.

```ruby
private_lane :master do
  lint_and_test 
  deploy_debug_version
end
```

I like how fastlane allows you to create such idiomatic CI flows. Look how small and declarative those lanes are. In reality, it's thanks to Ruby, the language in which fastlane is written in as is the `fastfile`.

First of all we lint and test the code and then we build a new version which gets subimtted to Fabric.

```ruby
private_lane :lint_and_test do
  set_version_number
  swiftlint # lint
  scan # test
end
```

When all the tests pass, we build a new version, deploy it to fabric, alert Slack and post a comment to Jira in the proper ticket with the build an version numbers so when someone goes into the ticket, he or she will know exactly which build to download. If your team has someone other than a developer testing apps, it will be a very big help and it will increase the productivity of the whole team.

```ruby
private_lane :deploy_debug_version do
  set_version_number
  match # install certificates
  gym # build
  submit_fabric
  post_to_slack
  post_to_jira
end
```
