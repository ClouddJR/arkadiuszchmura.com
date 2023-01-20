---
title: "How to fix the missing JDK problem on Electric Eel"
date: 2023-01-20
summary: When updating Android Studio to Electric Eel, the location of the embedded JDK is changing.
---

## Introduction

When updating your Android Studio to the newest Electric Eel version, the location of the embedded JDK is changed. In particular, it changes from:

`/Applications/Android Studio.app/Contents/jre/Contents/Home`

to

`/Applications/Android Studio.app/Contents/jbr/Contents/Home`

This might cause some problems with building your projects right after the update.

## Solution

First, make sure the JDK is correctly specified for your project. Go to `Preferences` -> `Build, Execution, Deployment` -> `Build Tools` -> `Gradle` and set the Gradle JDK to the correct one (if it didn't happen automatically):

{{< figure 
align=center 
src="/jdk.png" 
caption="Specifying JDK location in Preferences" 
>}}

Additionally, if your `JAVA_HOME` environment variable still points to the old location (as it was in my case), you will also have to update it.

> You can verify it by running `echo $JAVA_HOME` in your terminal. If it points to a completely different location, that's fine, you don't have to change that. It means you are using different JDKs in Android Studio and your terminal. However, I recommend using the same JDK everywhere to avoid problems with cache or running multiple Gradle daemons.

To update it, go to your `~/.zshrc` or `~/.bashrc` (or any other place where you define your variables) and make sure you have this line there:

`export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"`

## Summary

I hope this post helped you solve your problem. I wish the JDK location change had been signaled more clearly after the update so that we wouldn't have to spend time investigating all the issues ourselves.
