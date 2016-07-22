# JNI Native Bindings

Java Native Interfaces is the mechanism in which native C/C++ functions can be called as native methods in Java. It is the bridge between Java and C++. 

This part of the development process tends to be challenging. Debugging this area of the code is error-prone, and good JNI documentation and best practices are a bit lacking. Also, I've seen some pretty massive, monolithic JNI bridges that are many thousands of lines of code in a single file. Needless to say, one must exercise mindfullness to get this part right!

The code pertaining to the JNI bridge in Tangram is here:

https://github.com/tangrams/tangram-es/tree/master/android/tangram/jni

## Structure

![Android Bridge Relationship](images/android-bridge-relationship.png)

| Java | Bridge | C++ |
| -- | -- | -- |
| [MapController.java](https://github.com/tangrams/tangram-es/blob/master/android/tangram/src/com/mapzen/tangram/MapController.java) | [jniExports.cpp](https://github.com/tangrams/tangram-es/blob/master/android/tangram/jni/jniExports.cpp) | [platform_android.cpp](https://github.com/tangrams/tangram-es/blob/master/android/tangram/jni/platform_android.cpp) |

## Native Java Methods

All of the native methods JNI binds to are in [MapController.java](https://github.com/tangrams/tangram-es/blob/master/android/tangram/src/com/mapzen/tangram/MapController.java). The actual binding is in [jniExports.cpp](https://github.com/tangrams/tangram-es/blob/master/android/tangram/jni/jniExports.cpp), and the JNI setup is in [platform_android.cpp](https://github.com/tangrams/tangram-es/blob/master/android/tangram/jni/platform_android.cpp).

## Calling Java methods from C++

There are several places that Java methods are called from C++. All of this happens in [platform_android.cpp](https://github.com/tangrams/tangram-es/blob/master/android/tangram/jni/platform_android.cpp). Likewise, all of the Java methods that are called are in [MapController.java](https://github.com/tangrams/tangram-es/blob/master/android/tangram/src/com/mapzen/tangram/MapController.java).