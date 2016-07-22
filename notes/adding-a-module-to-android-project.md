# Adding a Module to Android Project

For some reason, whenever I try to add a module to an Android project in Android studio, I get the error:

```
Error:(1,) Plugin with id 'com.android.application' not found
```

It lets me `Open File`, which leads me to the module's `build.gradle`.

The solution is simple, just add the following block of code to the top of that `build.gradle` file:

```gradle
buildscript {
  dependencies {
    classpath 'com.android.tools.build:gradle:2.1.2'
  }
}
```

Of course, substitute whatever is the latest version of `build:gradle`.
