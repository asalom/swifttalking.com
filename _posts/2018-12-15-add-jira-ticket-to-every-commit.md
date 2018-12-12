---
layout: post
title: Add a Jira ticket number to each git commit
description: Most teams use some short of ticket system such as Jira or Trello. Learn how to add a JIRA ticket number to every git commit.
author: asalom
tags:   [tool]
---

Most teams use one tool to keep track of the issues to be solved, i.e. [Jira](https://www.atlassian.com/software/jira) and one tool to keep track of the code, i.e. [GitHub](https://github.com/). Linking stuff happening in one tool with the other can help us automate a lot of things. In a [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) or [Feature Branch](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) workflow each Jira ticket is solved in a different git branch, but how does Jira know in which branch it can find the implementation? And how does a git branch know to which Jira ticket it belongs?

Today we are going to learn how to link Jira issues to GitHub branches, commits and pull requests. 

### How to make Jira aware of GitHub and viceversa

The first step would be to [connect Jira to GitHub](https://confluence.atlassian.com/adminjiracloud/connect-jira-cloud-to-github-814188429.html). You might need to be an admin for this.

The second step would be to include the Jira issue number into each commit message, into each pull request title and into each branch name. Doing this manually sounds pretty annoying, right? If you're like me you'd forget about adding it always, specially to each commit. Let's automate this then. When I create a branch its normally named something like `feature/APP-123`, being `APP-123` a Jira issue. This helps me keep in the git history a reference to the Jira issue and to automate the inclusion of the issue number to each commit. I do this using a [git hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks).

> Git has a way to fire off custom scripts when certain important actions occur. There are two groups of these hooks: client-side and server-side. Client-side hooks are triggered by operations such as committing and merging, while server-side hooks run on network operations such as receiving pushed commits. You can use these hooks for all sorts of reasons.

