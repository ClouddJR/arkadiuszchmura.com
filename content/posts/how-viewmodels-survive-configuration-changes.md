---
title: How ViewModels survive configuration changes
date: 2022-04-26
summary: Let's explore their implementation details to see how they achieve this.
---


## Introduction

At Google I/O 2017, the Android Framework team introduced Architecture Components - a set of tools that significantly changed how many Android developers write and structure their apps. This post will focus on the internals of one of them - the [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel).

The `ViewModel` class is designed to store and manage UI-related data in a lifecycle conscious way. The `ViewModel` class allows data to **survive configuration changes** such as screen rotations.

Usually, one of the first things we find out when learning Android development is that activities get re-created after configuration changes. When it happens, we lose all initialized variables and the view gets re-rendered. We get a completely new activity instance.

So when I first heard about the `ViewModel` class and what it can do, some questions immediately popped up in my head: 

1. How is it possible that `ViewModels` survive configuration changes given that the activities that hold them get destroyed and created again? 
2. How does the newly created activity instance access a reference to the same `ViewModel` used by the previous activity instance?

Recently I decided to find answers to those questions. I chose to dive into the Android's source code and explore the `ViewModel`'s implementation details.

## Creating a ViewModel

Let's start from the beginning. The recommended approach (at the time of writing this post) to create an instance of a `ViewModel` class inside an activity is to use the following code:

```kotlin
private val viewModel: MyViewModel by viewModels()
```

where `by viewModels()` is a syntax representing [Kotlin delegated properties](https://kotlinlang.org/docs/delegated-properties.html). 

The `viewModels()` function returns a `Lazy<T>` instance, which serves as a lazy property delegate. This basically means that a `MyViewModel` instance is going to be obtained on first access to the `viewModel` variable (not when this variable is declared).

Here is what the `viewModels()` function looks like:

```kotlin
public inline fun <reified VM : ViewModel> ComponentActivity.viewModels(  
    noinline factoryProducer: (() -> Factory)? = null  
): Lazy<VM> {  
    val factoryPromise = factoryProducer ?: {
        defaultViewModelProviderFactory  
    }  
  
    return ViewModelLazy(VM::class, { viewModelStore }, factoryPromise)  
}
```

It accepts a single parameter - a `factoryProducer`. If it's specified, the `ViewModelProvider.Factory` returned by this lambda will be used to create a `ViewModel` instance. If not, the default one will be used. 

The function returns an instance of the `ViewModelLazy` class which is an implementation of the `Lazy` interface that I mentioned earlier.

The `ViewModelLazy`'s constructor takes three parameters. The first one represents a class of the `ViewModel` we want to create an instance of. The third one is a lambda that returns a `ViewModelProvider.Factory`. It's the same one as the one passed to the `viewModels()` function (or a default one). 

The second parameter is interesting. It's a lambda that returns a `ViewModelStore`. Here, a lambda is passed that returns a `viewModelStore` variable. Where is this variable coming from? 

As you can see, the `viewModels()` function is an extension function on the `ComponentActivity` class. So when we call `viewModelStore` in Kotlin, we effectively invoke the `getViewModelStore()` method from the `ComponentActivity` (written in Java) that returns its member variable called `mViewModelStore`:

```java
public ViewModelStore getViewModelStore() {
    if (getApplication() == null) {
        throw new IllegalStateException("Your activity is not yet attached to the "
                + "Application instance. You can't request ViewModel before onCreate call.");
    }
    ensureViewModelStore();
    return mViewModelStore;
}
```

The reason why the `ComponentActivity` has this method is that it implements the `ViewModelStoreOwner` interface. This is its declaration:

```java
/**
 * A scope that owns {@link ViewModelStore}.
 * <p>
 * A responsibility of an implementation of this interface is to retain owned ViewModelStore
 * during the configuration changes and call {@link ViewModelStore#clear()}, when this scope is
 * going to be destroyed.
 *
 * @see ViewTreeViewModelStoreOwner
 */
@SuppressWarnings("WeakerAccess")
public interface ViewModelStoreOwner {
    /**
     * Returns owned {@link ViewModelStore}
     *
     * @return a {@code ViewModelStore}
     */
    @NonNull
    ViewModelStore getViewModelStore();
}
```

Now you may ask: "What is `ViewModelStore`?"

As the name suggests, the `ViewModelStore` class is responsible for **storing** instances of `ViewModels`. This is what this class looks like:

```java
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    Set<String> keys() {
        return new HashSet<>(mMap.keySet());
    }

    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.clear();
        }
        mMap.clear();
    }
}
```

This relatively simple class serves as a wrapper around `HashMap<String, ViewModel>`. This is the ultimate place where all `ViewModels` associated with an activity or a fragment are stored.

Now, since we know what the `ViewModelStore` is, it's clear what the `ViewModelStoreOwner` interface is used for. A class that implements it indicates to the outside world that it owns an instance of the `ViewModelStore`.

This is a list of all classes in the Android framework that implement the `getViewModelStore()` method from the `ViewModelStoreOwner` interface. Basically, it's only activities and fragments.

{{< figure 
align=center 
src="/getViewModelStore.png" 
caption="Classes that implement `getViewModelStore()` from the `ViewModelStoreOwner`" 
>}}

Let's look at the documentation from the `ViewModelStoreOwner` interface, particularly this fragment:

> A responsibility of an implementation of this interface is to retain owned `ViewModelStore` during the configuration changes and call `ViewModelStore#clear()`, when this scope is going to be destroyed.

This gives us a valuable hint. It means that it's the **activity's (or fragment's) responsibility** to make sure that its `ViewModelStore` (along with all `ViewModels`) is preserved across configuration changes.

How do activities handle that? We will come back to this question in the next section.

For now, let's go back again to the `viewModels()` function and see what the newly constructed `ViewModelLazy` class looks like:

```kotlin
public class ViewModelLazy<VM : ViewModel> (
    private val viewModelClass: KClass<VM>,
    private val storeProducer: () -> ViewModelStore,
    private val factoryProducer: () -> ViewModelProvider.Factory
) : Lazy<VM> {
    private var cached: VM? = null

    override val value: VM
        get() {
            val viewModel = cached
            return if (viewModel == null) {
                val factory = factoryProducer()
                val store = storeProducer()
                ViewModelProvider(store, factory).get(viewModelClass.java).also {
                    cached = it
                }
            } else {
                viewModel
            }
        }

    override fun isInitialized(): Boolean = cached != null
}
```

Most of the code in this class just deals with caching the object and making sure that the same instance is returned on subsequent calls.

The most relevant fragment is this one:

```kotlin
ViewModelProvider(store, factory).get(viewModelClass.java)
```

It creates an instance of the `ViewModelProvider` class by passing the required parameters (that were created by executing the lambdas passed to the constructor) and calls `get()` to obtain our `ViewModel`.

Here's the `get()` method:

```kotlin
public open operator fun <T : ViewModel> get(modelClass: Class<T>): T {
    val canonicalName = modelClass.canonicalName
        ?: throw IllegalArgumentException("Local and anonymous classes can not be ViewModels")
    return get("$DEFAULT_KEY:$canonicalName", modelClass)
}
```

This methods call another `get()` method that additionally accepts a key as a parameter. In this case, the key is a concatenated string consisting of two parts separated by a colon:

* A `DEFAULT_KEY` value (which is `"androidx.lifecycle.ViewModelProvider.DefaultKey"`).
*  A canonical name of our `ViewModel` class. 

This key will be used to identify our `ViewModel` object in the `HashMap<String, ViewModel>` that we saw earlier in the `ViewModelStore` class.

Here's the second `get()` method:

```kotlin
public open operator fun <T : ViewModel> get(key: String, modelClass: Class<T>): T {
    var viewModel = store[key]
    if (modelClass.isInstance(viewModel)) {
        (factory as? OnRequeryFactory)?.onRequery(viewModel)
        return viewModel as T
    } else {
        @Suppress("ControlFlowWithEmptyBody")
        if (viewModel != null) {
            // TODO: log a warning.
        }
    }
    viewModel = if (factory is KeyedFactory) {
        factory.create(key, modelClass)
    } else {
        factory.create(modelClass)
    }
    store.put(key, viewModel)
    return viewModel
}
```

Without going into too many details, this method basically returns an existing `ViewModel` specified by the key (if there is one) or creates a new one of the desired type using the provided factory.

Here we can see the `ViewModelStore` in action. It's used to get an existing `ViewModel` instance (`var viewModel = store[key]`) or to store a newly created one (`store.put(key, viewModel)`). 

At this point, we've finally obtained the `ViewModel` instance that we wanted. It was either the one we already had access to or a completely new instance.

Ok, we’ve covered a lot of ground. It might be worth pausing for a moment to wrap up what we’ve found so far:

* Every activity and fragment (from the `androidx` packages) has a component called `ViewModelStore`. How do we know it? They declare this fact by implementing the `ViewModelStoreOwner` interface. The `ViewModelStore` has references to all `ViewModels` used by this activity or fragment. This component is preserved across configuration changes. Later in this post, we will see how it's done.
* When we call `private val viewModel: MyViewModel by viewModels()` in our activity (or fragment), we create a lazy property delegate that will initialize our `ViewModel` when we first access the `viewModel` variable. Internally, the code will create a correct instance and store it in the activity's (or fragment's) `ViewModelStore` or return the previous instance instead (if there was one).

## The survival

We learned that it's the `ViewModelStore` that stores our `ViewModels`. The original question: 

> How ViewModels survive configuration changes?

can therefore be rephrased to:

> How **ViewModelStores** survive configuration changes?

Let's focus on activities. Going back to the `getViewModelStore()` method from the `ComponentActivity`, we can notice that it calls another method called `ensureViewModelStore()` before returning its member instance.

As a refresher, here's the `getViewModelStore()` method:

```java
public ViewModelStore getViewModelStore() {
    if (getApplication() == null) {
        throw new IllegalStateException("Your activity is not yet attached to the "
                + "Application instance. You can't request ViewModel before onCreate call.");
    }
    ensureViewModelStore();
    return mViewModelStore;
}
```

and this is the `ensureViewModelStore()`:

```java
void ensureViewModelStore() {
    if (mViewModelStore == null) {
        NonConfigurationInstances nc =
                (NonConfigurationInstances) getLastNonConfigurationInstance();
        if (nc != null) {
            // Restore the ViewModelStore from NonConfigurationInstances
            mViewModelStore = nc.viewModelStore;
        }
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
    }
}
```

And there we go! It seems that we've found what we've been looking for. 

This method first checks if the `mViewModelStore` member variable is null. If it is, we **restore** the previous `ViewModelStore` (if there is any, otherwise, it creates a new one) using the `getLastNonConfigurationInstance()` method. This method returns an instance of the `NonConfigurationInstances` class that's defined as follows:

```java
static final class NonConfigurationInstances {  
    Object custom;  
    ViewModelStore viewModelStore;  
}
```

As we can see, it has our `ViewModelStore` object.

Now we know how our `ViewModelStores` are restored. There is one final piece of the puzzle to solve. We need to find out how they are saved. Let's start by looking at the documentation of the`getLastNonConfigurationInstance()` method:

> Retrieve the non-configuration instance data that was previously returned by `onRetainNonConfigurationInstance()`.

I think we are getting pretty close. Let's dive into the `onRetainNonConfigurationInstance()` method. First, this is what the documentation says about it:

> Called by the system, as part of destroying an activity due to a configuration change, when it is known that a new instance will immediately be created for the new configuration. You can return any object you like here, including the activity instance itself, which can later be retrieved by calling `getLastNonConfigurationInstance()` in the new activity instance.

And this is what the method looks like in the `ComponentActivity`:

```java
public final Object onRetainNonConfigurationInstance() {
    // Skipping the irrelevant parts...
	
    NonConfigurationInstances nci = new NonConfigurationInstances();
    nci.custom = custom;
    nci.viewModelStore = viewModelStore;
    return nci;
}
```

As you can see, this method prepares the instance of the `NonConfigurationInstances` class that will be retained across configuration change. It has our current `ViewModelStore` which means we will be able to successfuly restore it afterwards in the `getLastNonConfigurationInstance()`.

And that's it! All that seems so magical about `ViewModels` at first glance is just a combination of using this pair of methods from the `Activity` class:

* `onRetainNonConfigurationInstance()`
* `getLastNonConfigurationInstance()`

The process of saving and restoring the `ViewModelStore` in fragments is very similar. If you are interested, I encourage you to explore the source code to find it out by yourself.

## ViewModels and process death

Here's one thing worth keeping in mind. As mentioned in the introduction, the `ViewModel` class allows data to survive **configuration changes** such as screen rotations, enabling the multi-window mode, etc.

However, the system may destroy your application process while the user is away interacting with other apps. In such a case, the activity instance is destroyed, along with any state stored in it. This is called a *process death*. `ViewModels` **don't survive** a system-initiated process death.

This is why you should use `ViewModel` objects in combination with `onSaveInstanceState()` or some other disk persistence. To avoid some boilerplate when using the first approach, you might want to take a look at the [SavedStateHandle](https://developer.android.com/topic/libraries/architecture/viewmodel-savedstate).

## Summary

In order to effectively use `ViewModels` in our Android apps, it's not necessary to entirely understand the details of their implementation. However, many developers are simply curious about how some things work behind the scenes. Knowledge of the internals can make it easier to spot potential edge cases or pitfalls and simplify debugging in the future.

I hope you learned something new after reading this post and that it satisfied your curiosity. If there are still some parts that you find confusing, feel free to reach me on [Twitter](https://twitter.com/ClouddJR/) and ask some questions. I will do my best to answer them.