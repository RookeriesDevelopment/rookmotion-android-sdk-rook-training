# RookMotion Android SDK

RookMotion Android SDK allow new and existing applications to integrate RookMotion functionalities for training
measurement, tracking, and analysis.

RookMotion SDK is designed with the intention to be:

* Fast Integration: Useful to integrate fast and easy using pre-created views with RookMotion design and pre-automated
  flows. For reference: 4h to re-create the demo app.
* Full personalization: Better for apps with a dedicated development team, developers can call each SDK function in
  theirs desired order and create and fill their with a personalized UI.

These functionalities are:

* User's info, trainings, rewards, and statistics management in complement with the RookMotion API.
* Bluetooth's management for training sensors.

## Contents

1. [Installation](#installation)
2. [Configure SDK](#configure-sdk)
3. [RMClass](#rm)
    1. [DatabaseCleaner](#database-cleaner)
    2. [TrainingUploader](#training-uploader)
    3. [Preferences](#preferences)
    4. [Sensors](#sensors)
    5. [Trainings](#trainings)
    6. [TrainingTypes](#training-types)
    7. [Users](#users)
    8. [Centers](#centers)
    9. [Records](#records)
    10. [Metrics](#metrics)
4. [RMTrainer](#rmtrainer)
    1. [Methods](#rmtrainer-methods)
    2. [Callbacks](#rmtrainer-callbacks)
5. [RMBasalCalculator](#rmbasalcalculator)
    1. [Properties](#rmbasalcalculator-properties)
    2. [Methods](#rmbasalcalculator-methods)
    3. [Callbacks](#rmbasalcalculator-callbacks)
    4. [RelatedClasses](#rmbasalcalculator-related-classes)
6. [RMSensorScanner](#rmsensorscanner)
    1. [Methods](#rmsensorscanner-methods)
7. [RMSensorCentral](#rmsensorcentral)
    1. [RelatedClasses](#rmsensorcentral-related-classes)
8. [Other](#other)
    1. [Annotations](#annotations)
    2. [UTCTrainings](#utctrainings)
9. [Notes](#notes)
    1. [Licences](#licences)
    2. [DeleteAccount](#delete-account)

## Installation ##

### required software

To build a project using the RookMotion SDK you need to download and install

* [Android Studio](https://developer.android.com/studio) version 4.0 or later.

The Android recommended config in gradle is :

```groovy
android {
    compileSdkVersion 31

    defaultConfig {
        minSdkVersion 23
        targetSdkVersion 31
    }
}
```

RookMotionLink is available through gradle implementation. To implement it, simply follow the next steps:

1. Create a folder named **libs**.

2. Open your folder, drag and drop the given **aar** file into the **libs** folder

3. Add the following line to your build.gradle (project)

```groovy
allprojects {
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
}
```

or settings.gradle

```groovy
dependencyResolutionManagement {
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
}
```

4. Then declare your dependency on your build.gradle (app module):

```groovy
dependencies {
    implementation(name: 'rookmotionlink', ext: 'aar')
}
```

4. In the same dependencies scope add these lines

```groovy
dependencies {

    // RXJava
    implementation 'io.reactivex.rxjava3:rxandroid:3.0.0'
    implementation "com.squareup.retrofit2:adapter-rxjava3:2.9.0"

    // Retrofit
    implementation "com.google.code.gson:gson:2.8.9"
    implementation "com.squareup.retrofit2:retrofit:2.9.0"
    implementation "com.squareup.retrofit2:converter-gson:2.9.0"

    // OkHttp
    implementation "com.squareup.okhttp3:okhttp:4.9.0"
    implementation "com.squareup.okhttp3:logging-interceptor:4.9.0"
    testImplementation 'com.squareup.retrofit2:retrofit:2.9.0'
    testImplementation 'com.squareup.retrofit2:converter-gson:2.9.0'

    // Room
    kapt "androidx.room:room-compiler:2.4.0"
    implementation "androidx.room:room-ktx:2.4.0"
    implementation "androidx.room:room-runtime:2.4.0"
    implementation "androidx.room:room-rxjava3:2.4.0"

    // Timber
    implementation 'com.jakewharton.timber:timber:5.0.1'

    // Joda Time
    implementation 'joda-time:joda-time:2.10.6'
}
```

5. Add the following permissions to your AndroidManifest.xml

```xml

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.rookmotion">

    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

    <uses-permission
            android:name="android.permission.BLUETOOTH"
            android:maxSdkVersion="30"/>
    <uses-permission
            android:name="android.permission.BLUETOOTH_ADMIN"
            android:maxSdkVersion="30"/>

    <uses-permission
            android:name="android.permission.BLUETOOTH_SCAN"
            android:usesPermissionFlags="neverForLocation"
            tools:targetApi="s"/>

    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>

    <uses-feature
            android:name="android.hardware.bluetooth_le"
            android:required="true"/>
</manifest>
```

**Note:  It is important that: `android.hardware.bluetooth_le` option required is true. **

6. Sync your Gradle

## Configure SDK ##

We recommend using a **Service Locator** to keep a reference to the RM objects from every section of your app.

Create a package named rm

**Inside rm** create a class named RMServiceLocator with a context as a constructor parameter

```kotlin
class RMServiceLocator(context: Context)
```

Create a RMSettings, ApiLogger and RM object with lazy initialization

```kotlin
class RMServiceLocator(context: Context) {

    private val settings = RMSettings.getInstance().apply {
        setCredentials(
            "AUTH_TOKEN",
            "LEVEL_TOKEN",
            "https://www.google.com/", // Api core url
            "https://www.google.com/", // RookRemote url
            "https://www.google.com/", // Gamification url
        )
    }

    private val apiLogger = run {
        if (BuildConfig.DEBUG) {
            ApiLogger(HttpLoggingInterceptor.Level.BODY)
        } else {
            ApiLogger()
        }
    }

    val rm: RM by lazy { RM.getInstance(context, settings, apiLogger) }
}
```

Definitions:

| Parameter        | Description                                                                                                |
|------------------|------------------------------------------------------------------------------------------------------------|
| AUTH_TOKEN       | Auth key provided by Rookmotion                                                                            |
| LEVEL_TOKEN      | Client key provided by Rookmotion                                                                          |
| BASE_URL         | Backend url provided by Rookmotion                                                                         |
| REMOTE_URL       | Real time url provided by Rookmotion (if you don have one you can pass https://www.google.com/)            |
| GAMIFICATION_URL | Gamification backend url provided by Rookmotion (if you don have one you can pass https://www.google.com/) |

Create a class with the following convention YOUR_APP_NAME followed by Application, then make it extend from
Application()

```kotlin
class RookApplication : Application() {
    val rmServiceLocator by lazy { RMServiceLocator(this) }

    override fun onCreate() {
        super.onCreate()

        Timber.plant(Timber.DebugTree())
    }
}
```

In you AndroidManifest.xml add the following line on application tag

```xml

<application
        android:name=".RookApplication"/>
```

## RM ##

Helper class with RMApi and RMStorageManager methods. We strongly recommend using this class to facilitate development.

--------------------------------------------------

### Database cleaner ###

      doLogoutToDatabase(RMCallback callback)

Deletes all tables.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

      clearSummariesAndRecordsWhenEmptyTrainings(RMCallback callback)

Deletes all remaining summaries and records. This will execute when there are not pending trainings to upload.

We recommend using this method every time the app starts or in the callback of **uploadPendingTrainings(RMCallback,
boolean)**.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Training uploader ###

      finishAndUploadTraining(RMTrainingModel training, RMCallback callback)

Finishes the current training and starts upload process.

Parameters

**training** An RMTrainingModel instance with a valid start.

**callback** Execution to do after the request finishes, see RMResponse.

      uploadPendingTrainings(RMCallback callback, boolean deleteable)

Starts uploading finished trainings from Database to Api.

If you are using **clearSummariesAndRecordsWhenEmptyTrainings(RMCallback callback)** in the callback of this method you
must pass **deleteable as false**

Parameters

**deleteable** Pass true to delete trainings after a successful upload.

**callback**   Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Preferences ###

        getTrainingPreferences(RMGenericCallback<RMTrainingPreferences> callback)

Get current training preferences, see RMTrainingPreferences.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        saveTrainingPreferences(RMTrainingPreferences preferences, RMCallback callback)

Saves training preferences, if there is an existing training preferences object it will be replaced.

Parameters

**preferences** An RMTrainingPreferences instance with valid sensor and valid training type.

**callback** Execution to do after the request finishes, see RMResponse.

        updateLastUsedSensorInDatabase(RMSensor sensor, RMCallback callback)

Saves the provided sensor as last used. If an RMTrainingPreferences does not exist a new one will be created.

Parameters

**sensor**   An RMSensor instance with a valid UUID.

**callback** Execution to do after the request finishes, see RMResponse.

        updateLastUsedTrainingTypeInDatabase(RMTrainingType trainingType, RMCallback callback)

Saves the provided training type as last used. If an RMTrainingPreferences does not exist a new one will be created.

Parameters

**trainingType** An RMTrainingType instance with a valid UUID and name.

**callback** Execution to do after the request finishes, see RMResponse.

        deleteTrainingPreferences(RMCallback callback)

Deletes training preferences.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Sensors ###

        uploadPendingSensors(RMCallback callback)

Starts uploading sensors whose UUID is null from Database to Api.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getSensors(RMGenericCallback<List<RMSensor>> callback)

Get a list of user linked sensors from Api. Then saves the response on database. If Api request fails returns sensors
from Database.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getSensorsFromDatabase(RMGenericCallback<List<RMSensor>> callback)

Get a list of user linked sensors from Database.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        saveSensorFromBluetoothPeripheral(BluetoothPeripheral sensor, RMGenericCallback<RMSensor> callback)

Saves sensor in Api. If the sensor is already linked to a user 409 will be returned. If the request succeeds saves
sensor in Database.

Parameters

**sensor**   BluetoothPeripheral instance with a non-null MAC and name.

**callback** Execution to do after the request finishes, see RMResponse.

        saveSensorFromBluetoothDevice(BluetoothDevice sensor, RMGenericCallback<RMSensor> callback)

Saves sensor in Api. If the sensor is already linked to a user 409 will be returned. If the request succeeds saves
sensor in Database.

Parameters

**sensor**   BluetoothDevice instance with a non-null MAC and name.

**callback** Execution to do after the request finishes, see RMResponse.

        saveSensor(RMSensor sensor, RMGenericCallback<RMSensor> callback)

Saves sensor in Database. Then uploads to Api. If the sensor is already linked to a user 409 will be returned.

Parameters

**sensor**   RMSensor instance with a non-null MAC and name.

**callback** Execution to do after the request finishes, see RMResponse.

        saveSensorsInDataBase(List<RMSensor> sensors, RMCallback callback)

Saves sensors in database. If a sensor already exists, it will be replaced.

Parameters

**sensors**  List of rm sensors.

**callback** Execution to do after the request finishes, see RMResponse.

        deleteSensor(RMSensor sensor, RMCallback callback)

Deletes sensor from Api. If the request succeeds, deletes sensor from Database.

Parameters

**sensor**   RMSensor instance with a non-null UUID and Name.

**callback** Execution to do after the request finishes, see RMResponse.

        deleteSensorFromDatabase(String sensorName, RMCallback callback)

Deletes sensor from Database.

Parameters

**sensorName** Name of sensor to be deleted.

**callback** Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Trainings ###

        loadTraining(String startTime, GenericPairCallback<RMTrainingModel, RMSummary> callback)

Loads a pending training, and allows to continue it. This method is used in combination with
RMTraining#continueTraining(RMSummary, RMTrainingModel)**.

Parameters

**startTime** Training start time in the format YYYY-MM-DD HH:MM:SS, see **RMTrainingModel#getStart()**.

**callback**  Execution to do after the request finishes, see RMResponse.

        getTrainingsFromApi(String fromDate, String toDate, RMGenericCallback<List<TrainingListObject>> callback)

Get a list of user finished trainings from Api.

Parameters

**fromDate** Initial date in the format: YYYY-MM-DD.

**toDate**   Final date in the format: YYYY-MM-DD.

**callback** Execution to do after the request finishes, see RMResponse.

        getTrainingInformationFromApi(String trainingUUID, RMGenericCallback<RMTrainingInformation> callback)

Get information from a training from Api.

Parameters

**trainingUUID** Unique training identifier, see TrainingListObject#training_uuid.

**callback**     Execution to do after the request finishes, see RMResponse.

        getNotUploadedTrainings(RMGenericCallback<List<RMTrainingModel>> callback)

Get a list of user finished, but not uploaded trainings from Database.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getNotFinishedTrainings(RMGenericCallback<List<RMTrainingModel>> callback)

Get a list of user not finished trainings from Database.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        deleteUploadedTrainings(RMCallback callback)

Delete trainings that have been successfully uploaded.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        deleteTrainingFromApi(String trainingUUID, RMCallback callback)

Deletes a training from Api.

Parameters

**trainingUUID** Unique training identifier, see training_uuid on TrainingListObject.

**callback**     Execution to do after the request finishes, see RMResponse.

        deleteTrainingFromDatabase(String startTime, RMCallback callback)

Deletes the provided training from Database.

Parameters

**startTime** Start time of the training to delete in the format YYYY-MM-DD HH:MM:SS.

**callback** Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Training types ###

        getTrainingTypes(RMGenericCallback<List<RMTrainingType>> callback)

Get a list of available training types from Api. Then saves the response on database. If Api request fails returns
training types from Database.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getTrainingTypesFromDatabase(RMGenericCallback<List<RMTrainingType>> callback)

Get a list of available training types from Database.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getTrainingTypeExtrasFromApi(String trainingTypeUUID, RMGenericCallback<List<TrainingTypeExtra>> callback)

Get a list of custom attributes for a specific training type.

Parameters

**trainingTypeUUID** Unique training type identifier.

**callback**         Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Users ###

        getUserStatusFromApi(String email, RMGenericCallback<UserStatus> callback)

Checks user status and user UUID in Api.

Status codes description:

* 0 - User inactive
* 1 - User active

Parameters

**email** Valid email address of a registered user. If the email does not belong to a registered user 404 will be
returned.

**callback** Execution to do after the request finishes, see RMResponse.

        getUserFromApi(RMGenericCallback<RMUser> callback)

Returns user from api.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getUserFromApi(String userToken, String userUUID, RMGenericCallback<RMUser> callback)

Returns user from api.

Parameters

**userToken** User session token (RookLink's clients only need to provide an empty string)**.

**userUUID**  User unique identifier, can be obtained from RM#registerUserInApi(String, RMGenericcallback).

**callback**  Execution to do after the request finishes, see RMResponse.

        getUserFromDatabase(RMGenericCallback<RMUser> callback)

Returns user from Database.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getUserFromDatabase(String email, RMGenericCallback<RMUser> callback)

Returns user from Database filtered by provided email.

Parameters

**email**    User email.

**callback**  Execution to do after the request finishes, see RMResponse.

        getUserVariablesByPeriod(String fromDate,String toDate,String variable,RMGenericCallback<List<UserVariablesRecords>> callback)

Returns user variables history from api.

Parameters

**fromDate** Initial date in the format: YYYY-MM-DD.

**toDate**   Final date in the format: YYYY-MM-DD.

**variable** Example: "weight".

**callback** Execution to do after the request finishes, see RMResponse.

        registerUserInApi(String email, RMGenericCallback<String> callback)

Creates user on Api and returns its UUID, if the user already exists it only returns its UUID.

Parameters

**email**      To create an account from.

**callback**   Execution to do after the request finishes, see RMResponse.

        saveUserInDatabase(RMUser rmUser, RMCallback callback)

Saves a user to the Database, if a user with the same email exists this operation will replace the previous one.

Parameters

**rmUser**     Instance of RMUser.

**callback**   Execution to do after the request finishes, see RMResponse.

        updateUserProfile(RMUserAttributes rmUserAttributes,RMUserPhysiologicalVariables rmUserPhysiologicalVariables, RMCallback callback)

Updates user's RMUserPhysiologicalVariables and RMUserAttributes in Api. Then updates user in Database.

Parameters

**rmUserAttributes**             Instance of RMUserAttributes.

**rmUserPhysiologicalVariables** Instance of RMUserPhysiologicalVariables.

**callback**                     Execution to do after the request finishes, see RMResponse.

        updateUserAttributesInApi(RMUserAttributes rmUserAttributes, RMCallback callback)

Updates user's RMUserAttributes in Api.

Parameters

**rmUserAttributes** Instance of RMUserAttributes.

**callback**         Execution to do after the request finishes, see RMResponse.

        updateUserVariablesInApi(RMUserPhysiologicalVariables rmUserPhysiologicalVariables, RMCallback callback)

Updates user's RMUserPhysiologicalVariables in Api.

Parameters

**rmUserPhysiologicalVariables** Instance of RMUserPhysiologicalVariables.

**callback**                     Execution to do after the request finishes, see RMResponse.

        updateUserInDatabase(RMUser rmUser, RMCallback callback)

Updates a user in the Database.

Parameters

**rmUser**     Instance of RMUser.

**callback**   Execution to do after the request finishes, see RMResponse.

        syncUserIndexes(RMGenericCallback<RMUser> callback)

Fetch user indexes from Api, then saves in Database. If there are not indexes in Database default ones will be used. Be
sure to sync your indexes once in a while, specially when the user changes their RMUserPhysiologicalVariables.

Parameters

**callback**   Execution to do after the request finishes, see RMResponse.

        getIndexesFromDataBase(RMGenericCallback<UserIndexes> callback)

Returns user's UserIndexes from Database.You don't normally need these, they are used internally. Only be sure to update
them once in a while, specially when the user changes their RMUserPhysiologicalVariables.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        deleteUserFromDB(boolean deleteCache, RMCallback callback)

Deletes all users on Database.

Parameters

**deleteCache** Delete in memory cache, recommended to always pass true.

**callback** Execution to do after the request finishes, see RMResponse.

        deleteUserFromAPI(RMCallback callback)

Deletes user from Api.

**WARNING:** This will delete all user information, trainings and other data FOREVER.

After obtaining a successful response from this method you must call the following methods to clean up local data:

1. RM.doLogoutToDatabase(RMCallback)
2. RM.deleteUserFromDB(boolean, RMCallback) with deleteCache on true
3. Then you can navigate the user to your login screen.

**NOTE:** This does not actually delete data, more info on [DeleteAccount](#delete-account) section.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Centers ###

        getCenterByUUID(String centerUUID, RMGenericCallback<RMCenterObject> callback)

Get information of a center by its uuid in a RMCenterObject.

Parameters

**centerUUID** Unique identifier of the center

**callback**   Execution to do after the request finishes, see RMResponse.

        getCenterList(RMGenericCallback<List<RMCenterObject>> callback)

Get a list of the available centers, the returned list of RMCenterObject only contains basic information. To see all the
details of a center use RM#getCenterByUUID(String, RMGenericcallback).

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        getLinkedCenterList(RMGenericCallback<List<RMRegisteredCenter>> callback)

Get a list of the centers the user is currently rolled in.

Parameters

**callback** Execution to do after the request finishes, see RMResponse.

        isUserLinkedToBranch(String branchUUID, RMCallback callback)

Checks if user is linked to a branch.

Parameters

**branchUUID** Branch identifier. See **RMCenterObject#getBranchUUID()**.

**callback**   Execution to do after the request finishes, see RMResponse.

        linkUserToCenter(String branchUUID, RMGenericCallback<RMLinkBranchResponse> callback)

Enroll user to a center branch.

Parameters

**branchUUID** Branch identifier. See **RMCenterObject#getBranchUUID()**.

**callback**   Execution to do after the request finishes, see RMResponse.

        autoLinkUserToCenter(String branchUUID, RMGenericCallback<RMLinkBranchResponse> callback)

Enroll user to a center branch.

**WARNING:** This method won't send a request, the user will be auto accepted in the branch.

Parameters

**branchUUID** Branch identifier. See **RMCenterObject#getBranchUUID()**.

**callback**   Execution to do after the request finishes, see RMResponse.

        unlinkUserFromCenter(String branchUUID, RMCallback callback)

Unroll user from a center branch.

Parameters

**branchUUID** Branch identifier. See **RMCenterObject#getBranchUUID()**.

**callback**   Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Records ###

        getHeartRateRecordsFromDatabase(String startTime, RMGenericCallback<List<HeartRateDerivedData>> callback)

Gets all heart rate records of the provided date.

Parameters

**startTime** Start time of the records to find, in the format YYYY-MM-DD HH:MM:SS.

**callback**  Execution to do after the request finishes, see RMResponse.

        getStepsRecordsFromDatabase(String startTime, RMGenericCallback<List<StepsDerivedData>> callback)

Gets all steps records of the provided date.

Parameters

**startTime** Start time of the records to find, in the format YYYY-MM-DD HH:MM:SS.

**callback**  Execution to do after the request finishes, see RMResponse.

        getBicycleRecordsFromDatabase(String startTime, RMGenericCallback<List<RMBicycleRecord>> callback)

Gets all bicycle records of the provided date.

Parameters

**startTime** Start time of the records to find, in the format YYYY-MM-DD HH:MM:SS.

**callback**  Execution to do after the request finishes, see RMResponse.

        getSummariesFromDatabase(String startTime, RMGenericCallback<RMSummary> callback)

Gets all summaries of the provided date.

Parameters

**startTime** Start time of the summaries to find, in the format YYYY-MM-DD HH:MM:SS.

**callback**  Execution to do after the request finishes, see RMResponse.

        deleteRecordsAndSummaries(String startTime, RMCallback callback)

Deletes all heart rate records, steps records and summaries of the provided date.<br>
There will be 3 delete operations executing one after another, expect to receive 3 events in the callback's on finish
method.

Parameters

**startTime** Start time of the records/summaries to delete in the format YYYY-MM-DD HH:MM:SS.

**callback**  Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

### Metrics ###

        getTrainingsCountByPeriod(String fromDate, String toDate, RMSummaries.RMTRainingCountCallback callback)

Request the web service for user's training count between the provided dates.

Parameters

**fromDate** Initial date in the format: YYYY-MM-DD.

**toDate**   Final date in the format: YYYY-MM-DD.

**callback** Execution to do after the request finishes, see RMResponse.

        getSummariesByPeriod(String fromDate, String toDate, String summaryId, RMSummaries.RMSummaryByIdCallback callback)

Request the web service for a specific user summary (see summary list below)** between the provided dates. Summary ID
list:

1. Calories: 400
2. Heart rate: 101
3. Steps: 500
4. Time: 1

Parameters

**fromDate** Initial date in the format: YYYY-MM-DD.

**toDate**   Final date in the format: YYYY-MM-DD.

**callback** Execution to do after the request finishes, see RMResponse.

        getRewardsByPeriod(String fromDate, String toDate, RMSummaries.GetRewardsByPeriodCallback callback)

Request the web service for all user's rewards between the provided dates.

Parameters

**fromDate** Initial date in the format: YYYY-MM-DD.

**toDate**   Final date in the format: YYYY-MM-DD.

**callback** Execution to do after the request finishes, see RMResponse.

--------------------------------------------------

## RMTrainer ##

Helper class to do trainings.

Each instance of RMTrainer can do only one training.

If you want to do another you'll have to call **RMTrainer.onDestroy()** and create a new instance.

Constructor parameters

**context** Application, activity or fragment context.

### RMTrainer Methods ###

        onDestroy()

Disconnects from every connected sensor and releases bluetooth resources, call this when you no longer need this
instance.

        enableStepsReading(boolean enabled)

Enable steps notifications. If the sensor does not emit steps measurements, this won't do anything.

This method is not reversible.

Parameters

**enabled** True to enable notifications.

        enableBicycle()

Enables bicycle records insertion. This method is not reversible.

Parameters

**context** A Context instance to access bluetooth services.

        getStartTime()

Returns the date in witch the training started.

Returns

**String** A string with the format YYYY-MM-DD HH:MM:SS.

        getDuration()

Returns current training duration. To get automatic updates of this value use **RMTrainer.onTimeReceived(float)**.

Returns

**Float** The seconds passed from the start of the training.

        getTrainingType()

Returns

**RMTrainingType** Current training type.

        setTrainingType(RMTrainingType trainingType)

Sets the current training type, if the training is active the previous training type will be replaced with the new one.

Parameters

**trainingType** New training type of current training.

        isTrainingActive()

Checks current training state.

Returns

**Boolean** True if current training is active.

        isTrainingPaused()

Checks current training state.

Returns

**Boolean** True if current training is active, and paused.

        startTraining()

Starts training session.

**WARNING:** This must only be used once per RMTrainer instance.

        continueTraining(RMSummary summaries, RMTrainingModel training)

Continue a pending training.

**WARNING:** This is for pending trainings only and must not be used to resume a paused training.

To easily get the summaries and training model from a start time use **RM.loadTraining(String,
RMUtils.GenericPairCallback)**.

Parameters

**context** A Context instance to access bluetooth services.

        pauseTraining()

Pause the current training, to resume use **RMTrainer.resumeTraining()**.

        resumeTraining()

Resumes a training paused with **RMTrainer.pauseTraining()**.

        finishTraining()

Finishes the current training.

        finishAndUploadTraining()

Finishes the current training and starts upload process if this fails, you can try again using
**RM.uploadPendingTrainings(RMUtils.RMCallback, boolean)**.

        cancelTraining()

Cancels the current training, this will also delete training model, summaries, HR and Steps records from Database.

        isBluetoothEnabled()

Checks current bluetooth state.

Returns

**Boolean** True if bluetooth is enabled.

        startDiscovery(long timeout)

Starts sensor discovery, unless provided a valid timeout this call will never cancel until you call **RMTrainer
.cancelDiscovery()**.

**WARNING:** Please verify that your bluetooth is enabled before calling this.

Parameters

**timeout** Sets a timeout (in milliseconds) for the discovered process to be cancelled, must be greater than 1000.

        cancelDiscovery()

Cancels sensor discovery process.

**WARNING:** Please verify that your bluetooth is enabled before calling this.

        isDiscovering()

Check discovering state.

Returns

**Boolean** True if there is a discovery process running.

        getPeripheralFromMac(String mac)

Creates a BluetoothPeripheral from a sensor mac address.

This method will apply toUpperCase to the provided mac.

Parameters

**mac** Sensor mac address.

Returns

**BluetoothPeripheral?** A BluetoothPeripheral instance. Null if instance cannot be created.

        connectSensor(BluetoothPeripheral sensor)

Attempts to connect to a sensor then uploads the device whether the connection attempt was successful or not.

It is not necessary to scan for the peripheral first.

It's not necessary but recommended to make sure the device is cached first **BluetoothPeripheral.isUncached()**.

This attempt will not time out. To cancel you'll need to call **RMTrainer.cancelConnection(BluetoothPeripheral)**.

The upload process is described below:

1. BluetoothDevice is uploaded to server to get a UUID.
2. An RMSensor object is returned by the server and saved on sensors table.
3. The same RMSensor object is saved as last used sensor.

Parameters

**sensor** BluetoothPeripheral instance.

        connectLastUsedSensor(BluetoothPeripheral sensor, RMSensor rmSensor)

Attempts to connect to a sensor without uploading or saving.

It is not necessary to scan for the peripheral first.

It's not necessary but recommended to make sure the device is cached first **BluetoothPeripheral.isUncached()**.

This attempt will not time out. To cancel you'll need to call **RMTrainer.cancelConnection(BluetoothPeripheral)**.

Parameters

**sensor** BluetoothPeripheral instance.

**rmSensor** RMSensor instance, can be null. This is used to set current training sensor. If you don't provide one, the
last used sensor will be used. If there is no last used sensor. A generic sensor will be used.

        connectSensorWithTimeout(BluetoothPeripheral sensor)

Attempts to connect to a sensor then uploads the device whether the connection attempt was successful or not.

Make sure the device is cached first **BluetoothPeripheral.isUncached()**.

If the device is not in cache you must perform a discovery **RMTrainer.startDiscovery(long)**.

This attempt will time out in 30 seconds on most phones.

The upload process is described below:

1. BluetoothDevice is uploaded to server to get a UUID.
2. An RMSensor object is returned by the server and saved on sensors table.
3. The same RMSensor object is saved as last used sensor.

Parameters

**sensor** BluetoothPeripheral instance.

        connectLastUsedSensorWithTimeout(BluetoothPeripheral sensor, RMSensor rmSensor)

Attempts to connect to a sensor without uploading or saving.

Make sure the device is cached first **BluetoothPeripheral.isUncached()**.

If the device is not in cache you must perform a discovery **RMTrainer.startDiscovery(long)**.

This attempt will time out in 30 seconds on most phones.

Parameters

**sensor** BluetoothPeripheral instance.

**rmSensor** RMSensor instance, can be null. This is used to set current training sensor. If you don't provide one, the
last used sensor will be used. If there is no last used sensor. A generic sensor will be used.

        cancelConnection(BluetoothPeripheral sensor)

Cancels a connection attempt made only with **RMTrainer.connectSensor(BluetoothPeripheral)** or
**RMTrainer.connectLastUsedSensor(BluetoothPeripheral)**.

Parameters

**sensor** Sensor to cancel the connection attempt.

        finishSensorConnection()

Disconnects from current sensor.

        finishAllSensorsConnections()

Disconnects from every sensor.

        getConnectedSensor()

Retrieve current connected sensor instance.

Returns

**BluetoothPeripheral?** An instance of BluetoothPeripheral or null if no sensor is connected.

        addBicycleRecord(RMBicycleRecord rmBicycleRecord, boolean generateTimeStamp)

Save a new bicycle record linked to this training.

Parameters

**rmBicycleRecord**A RMBicycleRecord instance with a non-null power, cadence and resistance. Also, if generateTimeStamp
is set to false you have to provide a timestamp in the format YYYY-MM-DD HH:MM:SS.

**generateTimeStamp** If set to true a timestamp will be auto generated for this record.

### RMTrainer Callbacks ###

        onAdapterStateChanged(BluetoothAdapterState state)

Callback for Bluetooth adapter status changes. This can be triggered if the bluetooth is enabled/disabled.

Parameters

**state** Current bluetooth adapter status, see [BluetoothAdapterState](#bluetoothadapterstate---enum-class).

        onDiscoveredSensors(List<BluetoothPeripheral> sensors)

Callback for sensors discovered (Triggered each time a sensor is discovered).

Parameters

**sensors** List of discovered sensors (contains previous discovered sensors plus the new sensor).

        onNewSensorDiscovered(BluetoothPeripheral sensor)

Callback when a new sensor has been discovered with **RMTrainer.startDiscovery(long)** (Triggered each time a sensor is
discovered).

This will only return the recently discovered sensor, while **RMTrainer.onDiscoveredSensors(List)** will return a list
of the previous discovered sensors plus the new sensor.

Parameters

**sensor** Discovered sensor.

        onScanFailed(ScanFailure scanFailure)

Callback for BLE scan fails.

Parameters

**scanFailure** Failure reason.

        onConnectionStateChanged(SensorConnectionState state)

Callback for current connected sensor state updates.

Parameters

**state** Current bluetooth adapter status, see [SensorConnectionState](#sensorconnectionstate---sealed-class).

        onTimeReceived(float sessionSeconds)

Callback for current training duration (Triggered every second).

Parameters

**sessionSeconds** The seconds passed from the start of the training.

        onHrReceived(HeartRateDerivedData hrData)

Callback for HR data events.

Parameters

**hrData** Current HeartRateDerivedData measurements.

        onHrValidated(boolean isValid)

Checks if the raw HR received from connected sensor is valid. This only applies for active trainings.

Parameters

**isValid** To be considered valid an HR measurement must be grater than 0.

        onStepReceived(long currentSteps, float cadence)

Callback for steps events.

Parameters

**currentSteps** Current steps count.

**cadence** Current cadence.

        onBatteryLevelReceived(int level)

Callback for battery level (Triggered every second).

Parameters

**level** Current battery level of connected sensor.

        onBandContactReceived(RMBandContact bandContact)

Callback for the band contact state.

Parameters

**bandContact** Current band contact status, see RMBandContact.

--------------------------------------------------

## RMBasalCalculator ##

Helper class to obtain resting heart rate.

Constructor parameters

**durationInSeconds** Total duration of the calculation, new measurement is obtained every second, recommended duration
300 seconds.

**scope**             A CoroutineScope to execute calculations.

### RMBasalCalculator Properties ###

        bluetoothAdapterState

State flow that emits changes of the BluetoothAdapter, for more information
check [BluetoothAdapterState](#bluetoothadapterstate---enum-class).

        connectionState

State flow that emits changes of the current connected sensor, for more information
check [SensorConnectionState](#sensorconnectionstate---sealed-class).

        duration

State flow that emits current duration in seconds

        batteryLevel

State flow that emits battery level of connected sensor.

        sampleRegistry

List of all measurements obtained, this list will be cleared after the calculation finishes.

        running

Whether the calculation is active or not.

### RMBasalCalculator Methods ###

        onCreate(context: Context)

Initializes bluetooth.

Parameters

**context** A Context instance to access bluetooth services.

        onDestroy()

Clears all processes, ensure to invoke this when you no longer need to do any more calculations.

        start()

Starts basal calculation, the process will finish after the limit provided in _durationInSeconds_ or when you call stop.

        stop()

Stops basal calculation, this will also trigger _onBasalCalculationFinished_ to send whatever result was calculated at
the moment of calling this function.

Calling this function will reset the duration to the default provided in _durationInSeconds_ and clear up all the
previous HR samples.

        isBluetoothEnabled()

Checks current bluetooth state.

Returns

**Boolean** True if bluetooth is enabled.

        startDiscovery(timeout: Long)

Starts sensor discovery, unless provided a valid timeout this call will never be canceled until you call
_cancelDiscovery_.

**WARNING:** Please verify that your bluetooth is enabled before calling this.

Parameters

**timeout** Sets a timeout (in milliseconds) for the discovered process to be cancelled. Be sure to provide a value
greater than 1000.

        cancelDiscovery()

Cancels sensor discovery process.

**WARNING:** Please verify that your bluetooth is enabled before calling this.

        isDiscovering()

Check discovering state

Returns

**Boolean** True if there is a discovery process running.

        getPeripheralFromMac(mac: String?)

Creates a BluetoothPeripheral from a sensor mac address.

Parameters

**mac** Sensor mac address.

Returns

**BluetoothPeripheral?** A BluetoothPeripheral instance. Null if sensor cannot be created.

        connectSensor(sensor: BluetoothPeripheral)

Attempts to connect to a sensor.

It is not necessary to scan for the peripheral first.

It's not necessary but recommended to make sure the device is cached first _BluetoothPeripheral.isUncached_.

This attempt will not time out. To cancel you'll need to call _cancelConnection_

Parameters

**sensor** BluetoothPeripheral instance.

        connectSensorWithTimeout(sensor: BluetoothPeripheral)

Attempts to connect to a sensor.

Make sure the device is cached first _BluetoothPeripheral.isUncached_.

If the device is not in cache you must perform a discovery _startDiscovery_.

This attempt will time out in 30 seconds on most phones.

Parameters

**sensor** BluetoothPeripheral instance.

        cancelConnection(sensor: BluetoothPeripheral)

Cancels a connection attempt made only with _connectSensor_.

Parameters

**sensor** Sensor to cancel the connection attempt.

        finishSensorConnection()

Disconnects from current sensor.

        getConnectedSensor()

Retrieve current connected sensor instance.

Returns

**BluetoothPeripheral?** An instance of BluetoothPeripheral or null if no sensor is connected.

### RMBasalCalculator Callbacks ###

        onNewSensorDiscovered(sensor: BluetoothPeripheral)

Callback when a new sensor has been discovered with _startDiscovery_ (Triggered each time a sensor is discovered).

This will only return the recently discovered sensor, while _onDiscoveredSensors_ will return a list of the previous
discovered sensors plus the new sensor.

Parameters

**sensor** Discovered sensor.

        onDiscoveredSensors(sensors: List<BluetoothPeripheral>)

Callback for sensors discovered with _startDiscovery_ (Triggered each time a sensor is discovered).

Parameters

**sensors** Discovered sensors.

        onScanFailed(scanFailure: ScanFailure)

Callback for BLE scan fails.

Parameters

**scanFailure** Failure reason

        onSampleObtained(sample: RMRestingHearRateSample)

Callback when a _RMRestingHearRateSample_ has been obtained, this will be triggered every second.

**This is just for ui usages, it does not represent the final basal resting hear rate, that result will be delivered
through _onBasalCalculationFinished_**.

Parameters

**sample** Current resting heart rate sample.

        onBasalCalculationFinished(sample: RMRestingHearRateSample)

Callback for basal calculation results, this will only be triggered once after the provided _durationInSeconds_ has been
completed or if you called _stop_.

**WARNING:** This suspend method is called on background if you want to update an UI component in here, please change to
Main dispatcher.

Parameters

**sample** Final resting heart rate sample. To check if this is a valid result see
_RMRestingHearRateSample.basalResult_.

### RMBasalCalculator Related classes ###

#### BluetoothAdapterState - Enum class ####

This class describes bluetooth adapter states.

Values

**NONE**        Initial state, indicates that no interaction has been made.

**TURNING_OFF** Indicates that device's bluetooth adapter is turning off, this does not mean is already turned off, you
must wait for _OFF_ state.

**OFF**         Indicates that device's bluetooth adapter has been turned off.

**TURNING_ON**  Indicates that device's bluetooth adapter is turning on, this does not mean is already turned on, you
must wait for _ON_ state.

**ON**          Indicates that device's bluetooth adapter has been turned on.

#### SensorConnectionState - Sealed class ####

This class describes sensor connection states.

Values

**None**        Initial state, indicates that no sensor interaction has been made.

**Connecting** Indicates that connection process with a sensor has begun. **This does not mean that a connection has
been established**, you must wait for _Connected_ state.

* Properties
    * sensor - Sensor witch a connection attempt has been started.

**ConnectionFailed** Indicates that a connection attempt with a sensor has failed.

* Properties
    * status - Current HCI status.
    * sensor - Sensor witch a connection attempt was not successful.

**Connected**  Indicates that a sensor has been connected.

* Properties
    * sensor - Sensor witch connection has been established.

**Disconnecting** Indicates that disconnection process with a sensor has begun. **This does not mean that the current
connected sensor is disconnected**, you must wait for _Disconnected_ state.

* Properties
    * sensor - Sensor witch a disconnection attempt has been started.

**Disconnected** Indicates that current sensor has lost its connection.

* Properties
    * status - Current HCI status.
    * sensor - Sensor witch connection has been lost.

#### RMRestingHearRateSample - Data class ####

Class representation of a heart rate measurement.

Constructor parameters

**hearRate**    Current bpm.

**timeStamp**   Date of the measurement in the format YYYY-MM-DD HH:MM:SS.

**basalResult** Whether it is a valid result or not,
see [RMBasalCalculationResult](#rmbasalcalculationresult---enum-class).

#### RMBasalCalculationResult - Enum class ####

This class describes the result of a heart rate measurement.

This enum is returned with every RMRestingHearRateSample to indicate if the current measurement is a good candidate to
be considered as a resting heart rate measurement.

Values

**SUCCESS**         This measurement can be used as the resting heart rate.

**FAILED_TOO_LOW**  This measurement is too low and cannot be used as the resting heart rate. User may not be wearing
the sensor properly.

**FAILED_TOO_HIGH** This measurement is too low and cannot be used as the resting heart rate. User must be completely
relaxed to avoid this.

## RMSensorScanner ##

Helper class for scanning HR sensors.

Constructor parameters

**context** Application, activity or fragment context.

**sensorScannerListener** [RMSensorScannerListener](#rmsensorscannerlistener---interface) instance, this will be used to
receive sensors discovered and other bluetooth events.

### RMSensorScanner Methods ###

        isBluetoothEnabled()

Checks current bluetooth state.

Returns

**Boolean** True if bluetooth is enabled.

        startDiscovery()

Starts sensor discovery, this call will never be canceled until you call stopDiscovery().

**WARNING:** Please verify that your bluetooth is enabled before calling this.

        startDiscovery(long timeout)

Starts sensor discovery.

**WARNING:** Please verify that your bluetooth is enabled before calling this.

Parameters

**timeout** Sets a timeout (in milliseconds) for the discovered process to be cancelled, must be greater than 1000.

        stopDiscovery()

Stops sensor discovery process.

**WARNING:** Please verify that your bluetooth is enabled before calling this.

        isDiscovering()

Check discovering state.

Returns

**Boolean** True if there is a discovery process running.

        getPeripheralFromMac(String mac)

Creates a BluetoothPeripheral from a sensor mac address.

Parameters

**mac** Sensor mac address.

Returns

**BluetoothPeripheral** A BluetoothPeripheral instance. Null if instance cannot be created.

Throws

**IllegalArgumentException** If the mac is wrong.

**NullPointerException** If the mac is null.

        onDestroy()

Removes pending callbacks and releases bluetooth resources, ensure to call this when you finish using this class.

## RMSensorCentral ##

Helper class for scanning, connecting and obtaining measurements from HR sensors.

**IMPORTANT :** This class is already implemented internally. It's only listed here for documentation purposes.

### RMSensorCentral Related classes ###

#### RMSensorScannerListener - Interface ####

Implement this to receive sensors discovered and other bluetooth events.

Methods

**adapterStateChanged** Callback when Bluetooth adapter status changes. This can be triggered if the bluetooth is
enabled/disabled.

* Parameters
    * state - Current bluetooth adapter status, see BluetoothAdapter STATE constants.

**newSensor** Callback when a new sensor has been discovered (Triggered each time a sensor is discovered).

* Parameters
    * sensor - Discovered sensor.
    * scanResult - Result of BLE scan.

**newSensorAdded** Callback for sensors discovered (Triggered each time a sensor is discovered).

* Parameters
    * sensors - List of discovered sensors (contains previous discovered sensors plus the new sensor).

**scanFailed** Callback when BLE scan fails.

* Parameters
    * scanFailure - Failure reason.

#### RMSensorStatusListener - Interface ####

Implement this to receive sensor connection/disconnection events.

Methods

**connecting** Callback when a connection process with a sensor has begun. **This does not mean that a connection has
been established**, you must wait for _connected_ callback.

* Parameters
    * sensor - Sensor witch a connection attempt has been started.

**connectionFailed** Callback when a connection attempt with a sensor has failed.

* Parameters
    * sensor - Sensor witch a connection attempt was not successful.
    * status - Current HCI status.

**connected**  Callback when a sensor has been connected.

* Parameters
    * sensor - Sensor witch connection has been established.

**disconnecting** Callback when a disconnection process with a sensor has begun. **This does not mean that the current
connected sensor is disconnected**, you must wait for _disconnected_ callback.

* Parameters
    * sensor - Sensor witch a disconnection attempt has been started.

**disconnected** Callback when current sensor has lost its connection.

* Parameters
    * sensor - Sensor witch connection has been lost.
    * status - Current HCI status.

#### RMSensorMeasurementsListener - Interface ####

Implement this to receive measurements from a sensor.

Methods

**hrReceived** Callback for HR events.

* Parameters
    * bpm - Current user bpm.
    * bandContact - Current band contact status, see [RMBandContact](#rmbandcontact---enum-class).

**stepReceived** Callback for steps events.

* Parameters
    * currentSteps - Current steps count.
    * cadence - Current cadence.

**batteryLevelReceived** Callback for battery level.

* Parameters
    * level - Current battery level of connected sensor.

#### RMBandContact - Enum class ####

This class describes current contact state of a RookMotion band with user's body.

If user is not using a RookMotion band you can ignore these events.

Values

**CONTACT** Band has correct contact with users body.

**CONTACT_WITH_MEASUREMENTS** Band has correct contact with users body, and it's sending measurements.

**NO_CONTACT** Band has incorrect or no contact with users body

**NO_CONTACT_WITH_MEASUREMENTS** Band has incorrect or no contact with users body, but it's sending measurements.

**UNKNOWN** Could not validate contact type.

## Other ##

### Annotations ###

        @ExperimentalMethod

Indicates that current method is functional but not fully tested.

### UTCTrainings ###

From this SDK version onwards, all trainings timestamps will be on UTC timezone this includes:

* Start and End timestamps
* Summaries timestamps
* Records timestamps

When you request a list of trainings or the details of a training from server it will contain an additional property

      deviceOffset <- Available on TrainingListObject and RMTrainingInformation objects

If this property is null it means that this is NOT a UTC training, so you won't need to do any conversions.

On the other side if the property is non-null it will return a string with the offset the training was sent with, you
can use this offset to show the appropriate date to your end-users.

Although our recommendation is to only check if the property is non-null, then instead of using the returned offset,
with the help of a library (e.g. JodaTime), convert that training timestamp to the current device timezone.

```kotlin

// Full implementation on our demo app

object TimeUtils {
    fun getOffsetDateTime(date: String?, offset: String?): DateTime {
        return if (offset.isNullOrBlank()) {
            getDateTime(date)
        } else {
            getUTCDateTime(date).toDateTime(DateTimeZone.getDefault())
        }
    }
}

```

## Notes ##

### Licences ###

This documentation is provided by **RookMotion** all rights reserved to Rookeries Development SRL de CV.

**contact** : contacto@rookmotion.com

Startup MÃ©xico SUM Campus Ciudad de MÃ©xico

+52 55 39406461

---

RookMotionLink SDK uses BLESSED-BLE Library

See BLESSED-license.txt for more information.

---

### Delete Account ###

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
