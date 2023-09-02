# Android-Preparation

## Core building blocks of Android
- Activity
- Fragment
- Views
- Intent
- Service
- Content Providers
- Android Manifest

## Activity
#### Activity is a class that represents a single screen.
#### Activity has its own lifecycle method mentioned below.
- onCreate() - Called when an activity is first created.
- onStart() - Called when an activity becomes visible to the user.
- onResume() - Called when an activity starts interacting with the user.
- onPause() - Called when an activity is not visible to the user.
- onStop() - Called when an activity is no longer visible to the user.
- onRestart() - Called when an activity stopped, prior to restart.
- onDestroy() - Called when an activity is destroyed.

## Flow in Kotin
#### Flow is a reactive stream processing library that provides a way to emit and consumes streams of data asynchronously and efficiently.
![flow1.png](assets/flow1.png)
<img align="center" src="file:///assets/flow1.png" height="100" />

There are two types of Streams.
#### Hot Stream
Hot Stream produces data no matter consumer is consuming it or not.
#### Cold Stream
Cold Stream produces data only if consumer is consuming at the other side.





























