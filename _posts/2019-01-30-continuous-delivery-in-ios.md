---
layout: post
title: Continuous delivery in iOS
description: Building, testing and deploying our Apps is important because we'll be able to detect problems faster. I'll show you the flow that I've been using for a while.
author: asalom
tags:   [automation]
---

Keeping an App deployable at any time is very important for a number of reasons. Deploying the app very often somewhere [Fabric](https://fabric.io/), [TestFlight](https://developer.apple.com/testflight/), etc.) means that a version will be available for anyone to test at any time and if this version is always the last version it will reduce the friction between internal consumers of the App (PO, QA, CEO, etc) and developers. I've worked for some teams that didn't have a continuous delivery system in place and most weeks one of the developers had to manually install the latest version on someone's device. This meant a lot of hours lost into something that could be done automatically. If we ensure that the delivery method ([Fabric](https://fabric.io/), [TestFlight](https://developer.apple.com/testflight/), etc.) always contains the latest version, developers will have more time for what the do best: implement new features ~~and add new bugs~~.

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

Travis CI will only process a job if master is modified, or if we open a PR pointing to master. When this happens we'll execute `bundle exec fastlane travis` and fastlane will be deciding what flow to follow. Let's now see what happens inside the `fastfile`.

```ruby
lane :travis do
  if ENV['TRAVIS_PULL_REQUEST'] != 'false'
    pull_request
  elsif current_branch =~ /^master$/
    master # In our example, we'll execute this lane
  end
end
```

In this simplified example we can perform two flows, one when we open a Pull Request and the second one which we'll see in more detail, when master gets modified. Here we could add as many flows as we want. We could have one flow that will be executed every time we push something into the branch name `external/some_branch_name` and this will build and deploy a special version for our external testers.

So, this is what happens when master gets modified:

```ruby
private_lane :master do
  lint_and_test 
  deploy_debug_version
end
```

First of all we lint and test the code and then we build a new version which gets subimtted to Fabric.

I like how fastlane allows you to create such idiomatic CI flows. Look how small and declarative those lanes are. In reality, it's thanks to Ruby, the language in which fastlane is written. Everything we write into the `fastfile` is also Ruby.

```ruby
private_lane :lint_and_test do
  swiftlint # lint
  scan # test
end

private_lane :deploy_debug_version do
  set_build_number
  match # install certificates
  gym # build
  submit_fabric
  post_to_slack
  post_to_jira
end

private_lane :set_build_number do
  build_number = Time.new.strftime('%Y.%-m.%-d.%-H.%-M')
  increment_build_number build_number: build_number
end
```

When all the tests pass, we build a new version, deploy it to fabric, alert Slack and post a comment to Jira in the proper ticket with the build an version numbers so when someone goes into the ticket, he or she will know exactly which build to download. If your team has someone other than a developer testing apps, it will be a very big help and it will increase the productivity of the whole team by avoiding manual builds with Xcode.

The way to post into Slack and into Jira is pretty similar, I will explain only how to post a comment in a Jira ticket. The comment will contain the build and version numbers. First we are going to need a special Jira user so fastlane can log in and I highly recommend creating a special tech or bot user specifically for this purpose so we avoid having personal credentials managed by an automatic process.

For all of this to work we need to have the Jira ticket number somewhere inside the last commit message. This is how we'll know where to post the comment. [You can have a git-hook add the ticket number automatically for you in each commit](https://www.swifttalking.com/add-jira-issue-number-to-every-commit/).

```ruby
private_lane :post_to_jira do |options|
  last_commit = `git log --pretty=oneline -1`
  ticket = jira_ticket(from_text: last_commit)

  next unless ticket

  jira(
    url: 'https://my-jira.atlassian.net/',
    username: 'tech.user@example.com',
    password: 'password',
    ticket_id: ticket,
    comment_text: build_information
  )
end

def jira_ticket(from_text:)
  result = from_text.match /.*?(?<story>[A-Z]+-[0-9]+)/
  result[:story].to_s unless result.nil?
end

def build_information
  "Version: #{current_version}\nBuild: #{get_build_number}"
end

def current_version
  get_version_number(xcodeproj: 'Project.xcodeproj', target: 'Project')
end
```

The rest of the snippet is pretty much self explanatory. We get the last commit message contained in git, we try to extract the Jira ticket number from it and if we succeed we generate some text containing relevant information which we'll then place as a comment of the Jira ticket that we just merged.

We can see this in action in this screenshot taken from [crvsh' Jira](https://crvsh.com/).

<img width="600" alt="Flow" src="../images/posts{{page.url}}comment.png">