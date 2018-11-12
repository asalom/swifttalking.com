---
layout: post
title:  Generate animated GIFs from the iOS Simulator and the Android Emulator
description: Learn how to generate animtated GIFs from both the iOS Simulator and the Android Emulator. We can later attach this GIFs to Pull Requests or README files to showcase the running application.
author: asalom
tags:   [tool]
---

Here at [crvsh GmbH](https://crvsh.com/) I do pull requests on a daily basis. I also review them almost every day. While reviewing code from another colleague in GitHub’s Pull Request interface it is very hard to imagine how those changes will look like in a running App. This is why sometimes I even checkout my colleague’s branch and run it locally so I can see how everything looks like in the Simulator. 

![](../images/posts{{page.url}}gif-example.gif)

This has helped us achieve a higher level of quality in our product and at the same time it reduced drastically the amount of back and forth between us devs and them POs. Most of the tickets go through testing without any issues.

Sometime ago I worked developing the revamped version of [Welt News](https://itunes.apple.com/us/app/welt-news-nachrichten-live/id340021100?mt=8) app at [Welt.de](https://www.welt.de/). The team there had very extensive Pull Request reviews where we were validating each others code but we were not checking out those changes and testing them in the Simulator or in a device. This worked very well for us because we had a solid QA department that was finding even the smallest issue.

Sometime during that project we had a colleague who was accompanying the Pull Requests description with animated GIFs when the issue he has tackling had some short of animation or transition. Something that couldn’t be seen while checking his code. That was great, there was no need for a long description. You could just look at the GIF, check the code and understand how both worked together. I was curious on how my colleague Esad was producing such GIFs, I tried it but I didn’t find it very useful useful. At least not for that team.

As I mentioned, at crvsh we like running in the Simulator each other branches before approving a Pull Request so to make our work easier we started doing the same as my old colleague. Almost every Pull Request description where there is a UI change contains a GIF to facilitate the review. Recording a video in the iOS Simulator is pretty straight forward. You just have to run the following command:

```s
$ xcrun simctl io booted recordVideo recording.mp4
```

Now to convert that video to a GIF there are different options. Personally I liked using [😻 gifify](https://github.com/vvo/gifify). This was fine but it became very repetitive process: record video, stop video, execute gifify and remember the correct parameters.

## Capa

To overcome this repetitive process I decided to develop a utility called [capa which we recently open sourced](https://github.com/crvshlab/capa). capa lets you record a GIF and a video of your iOS Simulator all with one simple command: `$ capa-ios`. There is also another option to record the Android Emulator with `$ capa-android`

```s
$ capa-ios --help

> The command $ capa-ios generates a video and a GIF 
from the iOS Simulator.
> The command $ capa-android generates a video and a GIF
from the Android Emulator.

 Usage: capa-ios [options]
    -o, --output NAME         Output filename.
                              Defaults to 'recording'

    -v, --version             Display version
    -h, --help                Display help
```

Checkout the project at GitHub: [https://github.com/crvshlab/capa](https://github.com/crvshlab/capa)