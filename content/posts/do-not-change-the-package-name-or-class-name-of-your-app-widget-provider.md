---
title: Do not change the package name or class name of your AppWidgetProvider
date: 2022-02-22
summary: If you do this, widgets that were placed on the home screen by your users will simply disappear.
---

## Introduction

When creating a widget for your Android app, one of the components that you need to define is a class that extends [AppWidgetProvider](https://developer.android.com/reference/android/appwidget/AppWidgetProvider). Through it, you will receive broadcasts when the widget is updated, enabled, disabled, deleted, resized, etc. You can then act accordingly and, for example, refresh the widget's view to display the newest data.

I implemented a widget in [one of my apps](https://play.google.com/store/apps/details?id=com.arkadiusz.dayscounter) following the steps mentioned [in the documentation](https://developer.android.com/guide/topics/appwidgets) and everything was working just fine.

At least until one day. After noticing that my app is getting bigger, I decided to change the structure of my app a little to package classes by features rather than by layers.

When I changed the package name of my class extending AppWidgetProvider from 

```
com.arkadiusz.Provider.MyAppWidgetProvider
```

to 

```
com.arkadiusz.data.providers.MyAppWidgetProvider
``` 

and relaunched the app, all app's widgets disappeared from my home screen.

I can consider myself lucky for noticing this before releasing the next version of the app and causing this problem for all users.

It is quite strange that changing a package name of one class can produce such a radical effect so let's dive into the Android source code and find out why this is happening.

## Behind the scenes

In Android, a system service responsible for managing widgets across the entire OS is called [AppWidgetService](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/services/appwidget/java/com/android/server/appwidget/AppWidgetServiceImpl.java). This is a component that knows, among other things, **how** and **when** to update your widgets (based, for example, on the information that you specify in your AppWidgetProviderInfo XML). 

Internally, some of the variables that it holds are as follows:

```Java
private final ArrayList<Widget> mWidgets = new ArrayList<>();  
private final ArrayList<Host> mHosts = new ArrayList<>();  
private final ArrayList<Provider> mProviders = new ArrayList<>();
```

* **mWidgets** stores references to all widgets that are placed within all hosts.
* **mHosts** stores references to all [AppWidgetHosts](https://developer.android.com/reference/android/appwidget/AppWidgetHost), which are components designed to be used by applications that want to embed widgets in their UI (mostly home screen applications). They provide interaction with the AppWidget service.
* **mProviders** stores references to all AppWidgetProviders that were registered by currently installed apps. Each Provider object is identified by a [ComponentName](https://developer.android.com/reference/android/content/ComponentName) (which stores two pieces of information - package name and the class name). It also keeps references to all widgets that it is responsible for.

The AppWidget service also listens for different broadcasts related to application packages (like `Intent.ACTION_PACKAGE_ADDED` or `Intent.ACTION_PACKAGE_REMOVED`). It does that to keep its ArrayLists up to date. For example, when a user deletes an app from their device, the service can remove every widget and provider associated with this app because they are no longer needed.

I mentioned earlier that the service identifies each provider by its ComponentName. Well, if we decide to change the provider's package name or class name, the new ComponentName is going to be **different**. This is the root of the problem.

Here is what happens when you modify your provider's package name or class name and then update your app:

1. The AppWidget service is notified about an application package update via a broadcast.
2. It reads the app's manifest file and finds a new provider that it hasn't seen before (it has a unique ComponentName). 
3. It registers and stores it inside the **mProviders** variable.
4. Then, the service notices that it keeps a reference to a provider that is no longer used (your previous provider). There is no manifest file that declares it so the service decides that it is safe to **remove** it entirely. As part of the provider's removal process, every widget associated with that provider is removed as well.
5. You end up with empty widgets on the home screen that are no longer referenced by any component and cannot be updated.
6. The only way to get them to work again is to add them to the launcher anew.

### I am not alone

Interestingly, developers working on the Firefox app faced the same problem. If you look at the [source code](https://github.com/mozilla-mobile/fenix/blob/main/app/src/main/java/org/mozilla/gecko/search/SearchWidgetProvider.kt#L35) of their class extending AppWidgetProvider, you will find a comment at the top warning against modifying the package name or the class name:

```kotlin
class SearchWidgetProvider : AppWidgetProvider() {
	// Implementation note:
	// This class name (SearchWidgetProvider) and package name (org.mozilla.gecko.search) should
	// not be changed because otherwise this widget will disappear from the home screen of the user.
	// The existing name replicates the name and package we used in Fennec.
```

## Solution?

Unfortunately, there is currently no way to solve this problem as the code responsible for this behavior lies beyond our control. The only thing you can do is to make sure not to modify anything related to your widget provider after releasing your app if you don't want your users to lose their existing widgets.

Honestly, in most cases, it won't be a huge deal. Even though users would have to place those widgets again on their home screen, everything will continue to work as previously. However, if the configuration of widgets in your app is complicated and time-consuming, this can cause a lot of frustration.

## Conclusion

Once you implement widgets in your app and release it, do not change the package name or class name of your class extending AppWidgetProvider. It will wipe out all of your user's existing widgets and possibly bring a flood of 1-star reviews for your app.