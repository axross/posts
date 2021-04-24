---
title: Launching Texas Hold'em Poker Flutter App!
description: Submitted iOS/Android apps made of Flutter to app stores for the first time. It's a tool to calculate win rates in Texas Hold'em poker.
cover_image_url: https://user-images.githubusercontent.com/4289883/115946128-feebe980-a473-11eb-966c-0ebd54c3e569.png
tags:
  - flutter
first_published_at: '2019-11-07T00:00Z'
last_published_at: '2019-11-07T00:00Z'
---
I launched iOS/Android apps made of [Flutter](https://flutter.dev/) to app stores for the first time. It's a tool to calculate win rates for each hands or hand ranges up to 10 players.

![UI interaction](https://user-images.githubusercontent.com/4289883/115946161-2ba00100-a474-11eb-9556-d9bc73d00db6.png)

## Download

[:google-play](https://play.google.com/store/apps/details?id=app.axross.aqua&hl=en) [:app-store](https://apps.apple.com/us/app/odds-calculator-for-poker/id1485519383`)

This app is open source. **It would be really grateful if you leave a review at the app stores or star in the GitHub repository.**

[](https://github.com/axross/aqua)

## Implementation
Calculation process is just Monte Carlo method, which is trying randomly certain times and get average of that. In the real poker game, players guess what kind of cards the opponents have. This application is for such situation, so it's really impossible to calculate all possibilities because the time complexity will be O(n^m) at least.

Even the calculation in Monte Carlo method takes a lot of computation resource. You cannot do it in UI thread. So what I do is spawn a thread for calculation that communicates with UI thread. Flutter (if anything, Dart) has [Isolate](https://api.dartlang.org/stable/2.6.0/dart-isolate/dart-isolate-library.html), which is an abstract API for multi threading/processing. This app uses that.

## Was it hard to launch Flutter app?
No. But I recommend you to have experience of both of iOS and Android development. Coding part is well-wrapped by Flutter, therefore, you don't need any knowledge about Java, Swift, Xcode, Android and something like that. But in order to submit the app to the app stores, you will need to know about signing the app with a certification, minifying the app, building the app for each architecture of devices.

### Multithreading
Isolate is really easy to use. As long as you want to do some simple multithreading, you will suffer from nothing. Isolate has some restriction in good way, so that it is on safe and simple API, also stands as guideline like as Golang's Goroutine.

### Animation
Flutter supports sequence animations and tween animations.

![Flutter Tween Animation Example](https://videos.ctfassets.net/2mfcuy3p355s/4jPAo53HynNcYyE32mgJhV/1f83b17f483853f2bb903cd61ec05043/RPReplay_Final1574145129__2___1_.mp4) ![Flutter Tween Animation Example](https://videos.ctfassets.net/2mfcuy3p355s/5bER0UCiqkHJH1JyOx7f38/903960c488a9f719f6ab3063b51adfe4/RPReplay_Final1574145112__2___1_.mp4)

I didn't implement so complicated animation, but I feel animation in Flutter is much easier and more familiar than DOM's. Because Flutter's rendering is high performance and any property of UI is able to be used in animation.

### Dark mode
Flutter has [MediaQuery](https://api.flutter.dev/flutter/widgets/MediaQuery-class.html) API, which enables to obtain device information and OS settings such as:

- Display metrics and orientation
- Whether bold fonts are preferred (iOS)
- Whether 24-hour time format is used

Flutter has [InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html) that is same propagation model with [React's context](https://reactjs.org/docs/context.html). MediaQuery is on that. So you really easily can apply dark mode depending on users' OS setting to UI.

![Dark theme](https://user-images.githubusercontent.com/4289883/115946173-470b0c00-a474-11eb-8fb5-efd2029c6567.png)

## What Flutter isn't good for?

### Using ads
Flutter doesn't use platform's original UI framework at all. Instead of that, Flutter mounts a canvas to entire the screen and renders UI with its own rendering system. Therefore, sometimes Flutter's rendering performance exceeds native's one. However, as for ads, it's not really good with Flutter because usually they serves a SDK that is implemented on native UI framework.

### Using camera
Camera is also in the same situation. If you want to make camera apps such as Instagram or Snapchat, it's also quite hard. Because Flutter doesn't support camera by itself. In order to use camera, you'll call native API and it will rendered on native UI framework.
