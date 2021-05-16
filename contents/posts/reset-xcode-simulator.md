---
title: "How to reset Xcode Simulators"
date: "2020-10-26 18:40:27"
---

We often debug iOS/macOS application in simulators when developing iOS/macOS application. Sometimes we need a clean simulator environment, so we have to reset simulators.

Here's the steps:

## Step One

Shutdown all simulators manually, or execute command:

```shell
xcrun simctl shutdown all
```

## Step Two

Execute command:

```shell
xcrun simctl erase all
```

## That's it.
