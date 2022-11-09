# RookMotion Android SDK

RookMotion Android SDK allow new and existing applications to integrate RookMotion functionalities for training
measurement, tracking, and analysis.

RookMotion SDK is designed with the intention to be:

* Fast Integration: Useful to integrate fast and easy.
* Full personalization: Better for apps with a dedicated development team, developers can call each SDK function in
  theirs desired order and create and fill their with a personalized UI.

These functionalities are:

* User's info, trainings, rewards, and statistics management in complement with the RookMotion API.
* Bluetooth's management for training sensors.

## Contents

1. [Prerequisites](#prerequisites)
2. [Install](#install)
3. [Configure](#configure)
4. [Docs](#docs)
    1. [RM](#rm)
    2. [RookMotion](#rookmotion)
    3. [RMSensorScanner](#rmsensorscanner)
    4. [RMTrainer](#rmtrainer)
    5. [RMBasalCalculator](#rmbasalcalculator)
5. [Tutorials](#tutorials)
6. [Notes](#notes)
    1. [Licences](#licences)
    2. [DeleteAccount](#delete-account)

---

## Prerequisites

**IDE & Tools**

* Android Studio Chipmunk | 2021.2.1.
* Kotlin 1.6.10 (JVM Target 1.8).
* Android Device or Emulator with BLE capabilities and Android 6+ (sdk 23).

**Android SDK**

Go to your **build.gradle** (app level) and apply the following configuration.

```groovy
android {
    compileSdk 31

    defaultConfig {
        minSdk 23
        targetSdk 31

        // Other configurations
    }

    // More configurations
}
```

**Android Permissions**

Go to your **AndroidManifest.xml** and apply the following permissions.

```xml

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          package="com.rookmotion.rooktraining">

    <uses-permission android:name="android.permission.INTERNET"/>

    <!-- Optional for the sdk-utils library-->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS"/>

    <!-- Scan BLE peripherals -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

    <!-- Android 6+ (sdk 23) bluetooth permissions-->
    <uses-permission
            android:name="android.permission.BLUETOOTH"
            android:maxSdkVersion="30"/>
    <uses-permission
            android:name="android.permission.BLUETOOTH_ADMIN"
            android:maxSdkVersion="30"/>

    <!-- Android 12+ (sdk 31) bluetooth permissions-->
    <uses-permission
            android:name="android.permission.BLUETOOTH_SCAN"
            android:usesPermissionFlags="neverForLocation"
            tools:targetApi="s"/>
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>

    <!-- Filter your app from devices that do not have BLE capabilities. -->
    <uses-feature
            android:name="android.hardware.bluetooth_le"
            android:required="true"/>

    <!-- Other configurations -->
</manifest>
```

---

## Install

**Versioning**

This SDK uses Semantic Versioning: major.minor.patch.

Incompatible changes increment major version, adding compatible changes increment minor version, and compatible fixes
increment the patch version.

Each time a major release it made we'll provide you with a migration guide.

**Install from maven central**

Go to your **build.gradle** (app level) and declare the **sdk-rook-training** dependency, then sync your project.

* You can find the latest versions at [maven central](https://central.sonatype.dev/search?q=com.rookmotion.android).

```groovy
dependencies {

    // RookMotion
    implementation 'com.rookmotion.android:sdk-rook-training:$rt_latest_version'
    implementation 'com.rookmotion.android:sdk-utils:$u_latest_version' // Optional

    // Other dependencies
}
```

The **sdk-utils** library contains utilities for network state, permissions, etc. You can avoid installing it and
implement those utilities by yourself. The next tutorials will assume you installed it.

* **sdk-utils** includes joda-time library, version 2.10.6 as a compile dependency.

---

## Configure

To use our SDK you will need to initialize it every time your app launches. Follow the steps:

In your root package create a package named **rm**.

Create a class named **RMServiceLocator** with a context as a constructor parameter.

```kotlin
package com.rookmotion.rooktraining.rm

import android.content.Context

class RMServiceLocator(context: Context) {
}
```

Create a debug config in android > buildTypes of your **build.gradle** (app level) and add the following
buildConfigFields

Then in AndroidStudio's toolbar go to Build > Make project to generate a BuildConfig file.

```groovy
buildTypes {

    // Release configuration

    debug {
        debuggable true

        buildConfigField("String", "AUTH_TOKEN", "\"Bearer AUTH_TOKEN\"")
        buildConfigField("String", "LEVEL_TOKEN", "\"LEVEL_TOKEN\"")
        buildConfigField("String", "CORE_URL", "\"API_CORE_URL/api/v2/\"")
    }
}
```

Inside **RMServiceLocator** create an **RMSettings** instance, add your credentials and enable core features.

```kotlin
package com.rookmotion.rooktraining.rm

import android.content.Context
import com.rookmotion.rooktraining.BuildConfig

class RMServiceLocator(context: Context) {

    private val settings = RMSettings.getInstance().apply {
        addAuth(BuildConfig.AUTH_TOKEN, BuildConfig.LEVEL_TOKEN)
        enableCore(BuildConfig.CORE_URL)
    }
}
```

* Make sure your BuildConfig import is from your app package in this case is `com.rookmotion.rooktraining.BuildConfig`

Create an **RMApiSettings** instance, this will configure the requests logging and timeout options.

```kotlin
package com.rookmotion.rooktraining.rm

import android.content.Context
import com.rookmotion.rooktraining.BuildConfig

class RMServiceLocator(context: Context) {

    // RMSettings configuration

    private val apiSettings = run {
        if (BuildConfig.DEBUG) { // Only log when is a debug build
            RMApiSettings(ApiLogType.ADVANCED, 60000)
        } else {
            RMApiSettings(ApiLogType.NONE, 60000)
        }
    }
}
```

* Make sure your BuildConfig import is from your app package in this case is `com.rookmotion.rooktraining.BuildConfig`

Create an **RM** instance and a **RookMotion** instance.

```kotlin
val rm: RM by lazy { RM.getInstance(context, settings, apiSettings) }
val rookMotion: RookMotion by lazy { RookMotion.getInstance(context, settings, apiSettings) }
```

In your root package create a class with the following convention YOUR_APP_NAME followed by Application, this class must
extend from Application, then create an **RMServiceLocator** instance.

```kotlin
package com.rookmotion.rooktraining

import android.app.Application

class RookTrainingApplication : Application() {

    val rmServiceLocator by lazy { RMServiceLocator(this) }
}
```

Go to your **AndroidManifest.xml** in the `<application/>` tag, and add the previous class under the `name`
attribute.

```
<application
   android:name=".RookTrainingApplication"
   <!-- Other attributes -->
```

**Logging**

Rook Training SDK uses **RMApiSettings** to configure HTTP request logging, if you want to see logs for other
operations, like RMTrainer's measurements, you will need to add `Timber` dependency.

```groovy
dependencies {

    // Timber
    implementation 'com.jakewharton.timber:timber:5.0.1'

    // Other dependencies
}
```

And plant a debug tree.

```kotlin
import com.rookmotion.rooktraining.BuildConfig

class RookTrainingApplication : Application() {

    override fun onCreate() {
        super.onCreate()

        if (BuildConfig.DEBUG) { // Only log when in debug
            Timber.plant(Timber.DebugTree())
        }
    }
}
```

* Make sure your BuildConfig import is from your app package in this case is `com.rookmotion.rooktraining.BuildConfig`

---

## Docs

### RM

With this class you can have easy access to all **Java** features, it is a combination of low level classes and
resources (Database, Api, etc.)

You can access those resources by yourself but the **RM** class already takes care of complex operations like Database
and Api updates, Api response formatting, etc.

To explore the included functions go to
the [RM Javadoc](https://www.javadoc.io/doc/com.rookmotion.android/sdk-rook-training/latest/com/rookmotion/app/sdk/RM.html)

### RookMotion

The **RookMotion** class is the main dependency of all **Kotlin** features, which are divided in other **RMModules** (
RMRemote, RMSummaries, etc.), which use low level classes and resources (Database, Api, etc.)

You can access those resources by yourself but the **RookMotion** class, and it's modules already take care of complex
operations like; Database and Api updates, Api response formatting, etc.

To explore the included modules go to:

* [RookMotion Javadoc](https://www.javadoc.io/doc/com.rookmotion.android/sdk-rook-training/latest/com/rookmotion/kotlin/sdk/RookMotion.html)
* [RMSummaries Javadoc](https://www.javadoc.io/doc/com.rookmotion.android/sdk-rook-training/latest/com/rookmotion/kotlin/sdk/domain/core/RMSummaries.html)
* [RMRemote Javadoc](https://www.javadoc.io/doc/com.rookmotion.android/sdk-rook-training/latest/com/rookmotion/app/sdk/RMRemote.html)

### RMSensorScanner

With this class you can scan/discover sensors.

To explore the included functions go to
the [RMSensorScanner Javadoc](https://www.javadoc.io/doc/com.rookmotion.android/sdk-rook-training/latest/com/rookmotion/app/sdk/rmsensor/RMSensorScanner.html)

### RMTrainer

With this class you can connect to sensors and create training sessions.

* Each instance of RMTrainer can do only one training, If you want to do another you'll have to
  call `RMTrainer.onDestroy()` and create a new instance.

To explore the included functions go to
the [RMTrainer Javadoc](https://www.javadoc.io/doc/com.rookmotion.android/sdk-rook-training/latest/com/rookmotion/app/sdk/rmtrainer/RMTrainer.html)

### RMBasalCalculator

With this class you can connect to sensors and calculate your users resting heart rate.

To explore the included functions go to
the [RMBasalCalculator Javadoc](https://www.javadoc.io/doc/com.rookmotion.android/sdk-rook-training/latest/com/rookmotion/kotlin/sdk/domain/core/RMBasalCalculator.html)

---

## Tutorials

We have a step-by-step tutorial you can follow to create an MVP app of our SDK. Go to
our [Rook training docs](https://docs.rooklink.com/) for more info.

---

## Notes

### Delete Account

This section describes the procedure and impact of using either of the functions:

**deleteUserFromAPI** on **RM** Class

**deleteUserFromApi** on **RMUsers** Class

**deleteUser** on **RMUserApi** Interface

The previously listed methods are at RookMotionLink client's disposition to delete an account from RookMotion's servers.

Delete process consist of 3 steps:

1. Delete user's Email, Name, Pseudonym, PaternalSurname, MaternalSurname and Phone
2. Replace previously deleted data with other values.
3. Other information (trainings, weight, height, etc.) is keep on RookMotion's servers. But with the creator of that
   data as an anonymous user e.g. _fe1b1f6a@rookmotion.com_

Because this process is designed to completely anonymize an account it cannot be undone.

### Licences

This documentation is provided by **RookMotion** all rights reserved to Rookeries Development SRL de CV.

**contact** : contacto@rookmotion.com

Startup México SUM Campus Ciudad de México

+52 55 39406461

#### Third parties

Rook Training SDK uses BLESSED-BLE Library under the [MIT](https://choosealicense.com/licenses/mit/) licence.
