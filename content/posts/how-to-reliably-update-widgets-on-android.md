---
title: How to reliably update widgets on Android
date: 2022-02-12
summary: The default solution with android:updatePeriodMillis doesn't always work.
---

## Introduction

Users of [one of my apps](https://play.google.com/store/apps/details?id=com.arkadiusz.dayscounter) heavily rely on widgets. They place them on their home screens to count days to specific events. It is therefore important to keep those widgets up to date and refresh them at least once per day.

Not long ago, I have been getting some emails (as well as 1-star reviews) from my users reporting that their widgets are not being updated properly on their phones.

The Android documentation says that to update a widget periodically at some interval, you can specify `android:updatePeriodMillis` attribute on the widget's metadata - AppWigetProviderInfo XML. 

Unfortunately, despite setting this attribute in my app, widgets were still not updated reliably on some users' phones. 
 
The problem might be that some vendors [introduce their own set of rules](https://dontkillmyapp.com/problem) that restrict background processing and ignore this attribute altogether under some circumstances to artificially save some battery life.

## Solution

Since I could not count on this attribute alone, I needed another solution to support it. Luckily, Android Jetpack has a component that could help me periodically update widgets - WorkManager. As of writing this article, it is the primary recommended API for background processing. 

Quoting the documentation, WorkManager handles three types of persistent work:

* **Immediate**: Tasks that must begin immediately and complete soon. May be expedited.
* **Long Running**: Tasks which might run for longer, potentially longer than 10 minutes.
* **Deferrable**: Scheduled tasks that start at a later time and can run periodically.

Support for the third type of work perfectly matched my use case so I decided to give it a try.

Here are the steps I took to import and setup WorkManager in my app:

**1. Import the library into your Android project ([using the newest version](https://developer.android.com/jetpack/androidx/releases/work)):**

```gradle
dependencies {
    val work_version = "2.7.1"

    // Java only
    implementation("androidx.work:work-runtime:$work_version")

    // Kotlin + coroutines
    implementation("androidx.work:work-runtime-ktx:$work_version")
}
```

**2. Define the work**

Here we specify what our work actually **does**. We do this by creating our own implementation of the `Worker` class and overriding the `doWork()` method. This method runs asynchronously on a background thread provided by WorkManager.

In this case, we simply grab a list of relevant widget ids and notify the AppWidgetManager to perform an update on them by sending an Intent.

```kotlin
class WidgetUpdateWorker(
    private val appContext: Context,
    workerParams: WorkerParameters
) : Worker(appContext, workerParams) {

    override fun doWork(): Result {
        // This line might be different in your case
        val widgetIds = DatabaseRepository.getWidgetIds()

        val intent = Intent(
            AppWidgetManager.ACTION_APPWIDGET_UPDATE, null, appContext,
            AppWidgetProvider::class.java
        )
        intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_IDS, widgetIds)
        appContext.sendBroadcast(intent)

        return Result.success()
    }
}
```

One thing to note here - `MyAppWidgetProvider` is a subclass of the `AppWidgetProvider` that is needed when creating a widget.

**3. Define WorkRequest and schedule it**

Now that we defined our work, it is necessary to specify **how and when** that work should be run. We do this by defining a `WorkRequest`. In our case, we simply want to run the work periodically at some interval.

```kotlin
val widgetUpdateRequest = PeriodicWorkRequestBuilder<WidgetUpdateWorker>(
    4, TimeUnit.HOURS
).build()
```

Finally, we can put everything together and schedule our work. The code below could be placed in the `Application.onCreate()` or in your main activity's `onCreate()`.

```kotlin
WorkManager.getInstance(this).enqueueUniquePeriodicWork(
    "widgetUpdateWork",
    ExistingPeriodicWorkPolicy.KEEP,
    widgetUpdateRequest
)
```

The first parameter - `"widgetUpdateWork"` is a unique name that identifies the work.

The `ExistingPeriodicWorkPolicy.KEEP` flag tells the WorkManager to keep using the existing work if there is one already scheduled with the same name.

The third parameter is a `WorkRequest` that we've just defined.

And that's it! The work is now scheduled and our widgets are going to be periodically updated.

As a side note, there are many benefits to using the WorkManager versus relying on the standard way of updating the widgets. Here are some of them:

* Once you specify the `android:updatePeriodMillis` attribute, it is fixed and cannot be changed in any other way than updating the app with a new value. On the other hand, you can let your users decide how often widgets should be updated with WorkManager and change this value at runtime.
* WorkManager automatically ensures that the scheduled tasks are going to finish even when the OS decides to kill the app to reclaim some memory.
* Internally, WorkManager uses JobScheduler (on API 23+) or a combination of AlarmManager and BroadcastReceiver on older versions. This makes it less likely to be restricted by some vendors.

After introducing WorkManager, the number of emails from my users about their widgets not being updated properly dropped significantly.

## Conclusion

To summarize, here are the key points from this post:

* Some vendors restrict background processing in order to save some battery life. Because of this, the standard way of updating widgets (with the `android:updatePeriodMillis` attribute) doesn't always work.
* To solve this problem, we can introduce WorkManager that will periodically update widgets and make sure that they are up to date.