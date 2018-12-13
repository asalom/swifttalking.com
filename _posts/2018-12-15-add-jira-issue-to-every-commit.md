---
layout: post
title: Add a Jira issue number to each git commit
description: Most teams use some short of ticket system such as Jira or Trello. Learn how to add a JIRA ticket number to every git commit.
author: asalom
tags:   [tool]
---

Most teams use one tool to keep track of the issues to be solved, i.e. [Jira](https://www.atlassian.com/software/jira) and one tool to keep track of the code, i.e. [GitHub](https://github.com/). Linking stuff happening in one tool with the other can help us automate a lot of things. In a [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) or [Feature Branch](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) workflow each Jira ticket is solved in a different git branch, but how does Jira know in which branch it can find the implementation? And how does a git branch know to which Jira ticket it belongs?

Today we are going to learn how to link Jira issues to GitHub branches, commits and pull requests. 

### How to make Jira aware of GitHub and viceversa

The first step would be to [connect Jira to GitHub](https://confluence.atlassian.com/adminjiracloud/connect-jira-cloud-to-github-814188429.html). You might need to be an admin for this.

The second step would be to include the Jira issue number into each commit message, into each pull request title and into each branch name. Doing this manually sounds pretty annoying, right? If you're like me you'd forget about adding it always, specially to each commit. Let's automate this then. When I create a branch its normally named something like `feature/APP-123`, being `APP-123` a Jira issue. This helps me to keep in the git history a reference to the Jira issue and to automate the inclusion of the issue number to each commit. I do this using a [git hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks).

> Git has a way to fire off custom scripts when certain important actions occur. There are two groups of these hooks: client-side and server-side. Client-side hooks are triggered by operations such as committing and merging, while server-side hooks run on network operations such as receiving pushed commits. You can use these hooks for all sorts of reasons.

There are many git hooks but the one we are interested in for this purpose is called `commit-msg`.

> The commit-msg hook takes one parameter, which again is the path to a temporary file that contains the commit message written by the developer. If this script exits non-zero, Git aborts the commit process, so you can use it to validate your project state or commit message before allowing a commit to go through. In the last section of this chapter, We’ll demonstrate using this hook to check that your commit message is conformant to a required pattern.

All the client side hooks are placed inside `.git/hooks` and by default every repo comes with a disabled sample for each hook.

![](../images/posts{{page.url}}terminal-ls.png)

Now, to enable hook we just need to have a file inside `.git/hooks` with the right name, `commit-msg` in our case, and the execute permissions.

```s
$ cd .git/hooks
$ touch commit-msg
$ chmod +x commit-msg
```

The content of this file will a script that will try to find a Jira issue number from the branch name and if it does find it it will prepend it to each one of our commit messages. If a Jira issue number is not found, it will leave the commit message intact.

```s
#!/bin/sh

COMMIT_FILE=$1
COMMIT_MSG=$(cat $1)
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
JIRA_ID=$(echo "$CURRENT_BRANCH" | grep -Eo "[A-Z0-9]{1,10}-?[A-Z0-9]+-\d+")

if [ ! -z "$JIRA_ID" ]; then
  echo "[$JIRA_ID] $COMMIT_MSG" > $COMMIT_FILE
  echo "JIRA ID '$JIRA_ID' prepended to commit message. (Use --no-verify to skip)"
fi
```

Now if you just remember to add the Jira issue number of the ticket you're working on to your branch name, this script will add the Jira issue identifier to each one of your commit messages and if you also [connected Jira to GitHub](https://confluence.atlassian.com/adminjiracloud/connect-jira-cloud-to-github-814188429.html) you'll be able to see some information from GitHub inside the Jira issue.

![](../images/posts{{page.url}}jira-issue.png)