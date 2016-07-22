# Installation

Fortunately installation is remarkably easy with Tangram ES. I'm going to start with installing the demo app that is part of the Android component of Tangram ES.

## Building From Source

https://github.com/tangrams/tangram-es/tree/master/android

If you are planning on doing actual Tangram ES development in C++/GLSL, this is the first place for you to look, regarding getting the app working on an Android phone. The Mapzen Android SDK is important as well, however, it treats the underlying rendering library as an SDK, so you would indeed have to go in and specifically create an `aar` or gradle package each time you make a change to Tangram ES to see what you have done in your Android app.

Follow the Android build instructions in [README.md](https://github.com/tangrams/tangram-es#android). As noted, you'll need to setup your `ANDROID_HOME` and `ANDROID_NDK` environment variables. For me, the following is what I set:

    export ANDROID_SDK=/Users/njh/Library/Android/sdk
    export ANDROID_NDK=/usr/local/Cellar/android-ndk/r10e

Note that I've had best results so far installing the NDK with homebrew.

Then:

```
make android
```

Go ahead and execute `./android/run.sh`, and you should see the Tangram ES demo app open on your phone (if it is connected with all the development USB debug setup done). This is great, but we really want to have this working in Android Studio, and we want to be able to debug the Java.

When opening Android Studio, select __Import project (Eclipse ADT, Gradle, etc.)__. Choose the [`tangram-es/android/`](https://github.com/tangrams/tangram-es/tree/master/android) directory. That should be all you need to do. You can build and run the demo app from Android Studio.

Note: Android Studio complained that the gradle version was out of date. Allow Android Studio to update your gradle build version--it works fine!

```gradle
 buildscript {
   dependencies {
-    classpath 'com.android.tools.build:gradle:2.1.0'
+    classpath 'com.android.tools.build:gradle:2.1.2'
   }
 }
```

