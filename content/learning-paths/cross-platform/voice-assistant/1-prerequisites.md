---
title: Prerequisites
weight: 3

### FIXED, DO NOT MODIFY
layout: learningpathall
---

## Install software for this Learning Path

In this learning path, you will build an android application, so you first need
to download and install [Android Studio](https://developer.android.com/studio)
on your computer.

You then need to ensure you have the following tools:
- ``git``, the version control system, for cloning the Voice Assistant code base
- ``adb``, the Android Debug Bridge, a command line tool to communicate with a
  device and perform various commands on it.

by running the following command (depending on your machine's OS):

{{< tabpane code=true >}}
  {{< tab header="Linux/Ubuntu" language="bash">}}
sudo apt install git adb
  {{< /tab >}}
  {{< tab header="macOS" language="bash">}}
brew install git android-platform-tools
  {{< /tab >}}
{{< /tabpane >}}
