# Android-workmanager

##### What is WorkManager? What is WorkManager?
App won't always be in the foreground but you still need to do important background work like downloading, updates and synching with your server. now there are many existing api's for this, each with their own particular use cases. unfortunately when these api's are used incorrectly we see this low battery power for users. 

Android has introduced many battery saving features in recent releases. these features manage and balance the power usage of apps, as a developer you no need to work with these battery saving features to ensure background tasks run across different API levels. Maintaining different apis level adds code complexity, moreover if you don't do this properly you risk your background work not running for all users, this is where the work manager library comes in.

Work manager provides a unified solution for background work taking androids power saving features and the users API level into account. it's backwards compatible to API level 14 part of android jetpack and it runs with or without Google Play services. It the recommended solution for most background work on Android. 

There are two key attributes that make a task suitable for work manager 

1. Deferrable work
2. Guaranteed work

Deferrable work is any task which is still useful even if it's not run immediately. sending analytic data to your server is totally deferral work but sending an instant message is not deferrable.

Guaranteed work means that the task will run even if the app process is killed or the device restarts, so a great use case for work manager would be backing up pictures to a server, this work can be deferred but it should also be guaranteed. 

Work manager can be paired with other api's like firebase Cloud messaging to trigger background. for example when there's data ready to sync on your server you can use an FCM message to notify your app and then do the actual syncing with work manager.

Work manager is not a replacement for calling co-routines, thread pools or libraries like Rx java.

Work manager is also not designed to trigger work at an exact time, for that you're going to need to use an API like alarm manager or start a foreground service if your users expect your task to happen immediately

#### Work request
Let's say that I want to upload a photo using work manager. first you define what your task does using a worker like this one all the code goes into the do work method then you make a request to do this work using a class called work request 

```kotlin
class UploadWorker(appContext: Context, workerParams: WorkerParameters)
    : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        // Do the work here--in this case, upload the images.

        uploadImages()

        // Indicate whether the task finished successfully with the Result
        return Result.success()
    }
}


```

#### Adding constraints
As part of the work request you can add constraints specify the input and choose to run the work once or periodically. the work request is basically a class for configuring when and how your task will be run 

Below example is non-repeating work, one-time work request I've added a constraint here that it should only do this upload when there's actually Network. I know that this photo will be uploaded at some point of the future when the network constraint is met even if the app is closed or the phone restarts. 

```kotlin
// Create a Constraints object that defines when the task should run
val constraints = Constraints.Builder()
        .setRequiresNetworkType(NetworkType.Connected)
        .build()
        
val photoData = workDataOf(Contants.PHOTO_URI tp uri)
        
 val uploadWorkRequest =OneTimeWorkRequestBuilder<UploadWorker>()
        .setConstraints(constraints)
        .setInputData(photoData)
        .build()

        
WorkManager.getInstance().enqueue(uploadWorkRequest)
 ```

Now let's say that I want option of updated the UI when my work finishes, I can use the method get work info by ID live data. work info has all of the information about the current state of my work. 

```kotlin
WorkManager.getInstance().getWorkInfoByIdLiveData(uploadPhotoRequest.id)
```

#### Behind the scenes
Work manager chooses between job scheduler and a combination of alarmmanager and broadcast receivers. when running to guarantee that the work will keep running all of that information about a queued work is kept in a work manager bandaged database so that I can be resumed if it's ever stopped. 

By defult, work manager does its work off the main thread using executor behind the scenes. 

#### chains of dependent work
Work manager truly shines is when you create chains of dependent work, let's say that I want to add a filter to multiple pictures compress those pictures together and then batch upload the compressed file to share with the world assuming you created all of your work



Below link helps to learn more on it
https://codelabs.developers.google.com/codelabs/android-workmanager/#0
