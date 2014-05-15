---
layout: post
title: Building Android with Ant on the Command Line
---

Every Android developer has to build his project on the command line at some point. Building with Ant is very basic: in a single command you can build your project, run it through Proguard, and sign it with the keystore of your choice.

## Prerequisites

### Prep Libraries

Each library project that your app relies on must contain a `build.xml`. Without this your build will fail with the following:

    BUILD FAILED
    <path-to-your-project>/build.xml:110: The following error occurred while executing this line:
    <path-to-your-project>/build.xml:41: The following error occurred while executing this line:
    <path-to-your-android-sdk>/tools/ant/build.xml:515: Invalid file: <path-to-dependent-library-project>/build.xml

The fix is simple: `cd` into the library project and run the following:

    android update lib-project

*Note: The `android` executable is in your `android-sdk/tools` directory, which should be on your path. If you need to set this up, there are [plenty](http://spring.io/guides/gs/android/) [of](http://www.talkandroid.com/guides/developer/android-sdk-install-guide/) [tutorials](https://www.youtube.com/watch?v=k4FWDkzR7DA).*

### Keystore ###

Before you can build with Ant you will need to know what keystore you want to use. By default your keystore will be located in your user's home directory:

    ~/.android/debug.keystore

If you are part of a team of developers building the app, chances are you will have a shared keystore. If not, you can create one:

    keytool -genkey -v -keystore debug.keystore -storepass android -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 -validity 10000

### Build! ###

#### Debug

Running the debug task will not execute Progaurd.

    ant clean debug \
    -Dsdk.dir=<PATH-TO-ANDROID-SDK> \
    -Dkey.store=<PATH-TO-KEYSTORE>

#### Release ####

Running the release task will execute proguard if it is enabled.

    ant clean release \
    -Dsdk.dir=<PATH-TO-ANDROID-SDK> \
    -Dkey.store=<PATH-TO-KEYSTORE> \
    -Dkey.alias="<ALIAS>" \
    -Dkey.store.password=<STORE-PASSWORD> \
    -Dkey.alias.password=<ALIAS-PASSWORD>

### Further Reading ###

If you would like more information on Android's use of keystores or the release process, have a look at Google's documentation on [Signing Your Applications](http://developer.android.com/tools/publishing/app-signing.html).
