---
title: Be careful when converting Flow to LiveData
date: 2022-12-31
summary: LiveData created this way will only emit data when it has active observers.
---

## Introduction

Recently, I've been working on a codebase where I had to write a bridging code between a data layer using `Flows` and a UI layer that still relied on the state exposed as `LiveData`.

Luckily, there is a function in the `androidx.lifecycle` called `asLiveData()` that allows you to convert a `Flow` to a `LiveData` effortlessly. However, there is one caveat to keep in mind when using a `LiveData` created this way. It will only emit data when it has at least one active observer. If there is an update to the upstream flow and the `LiveData` is inactive, it will not have the latest value.

Let me show you a potential problem we might encounter, with an example below:

## Example

We have a simple activity that keeps a reference to an AAC `ViewModel`:

```kotlin
class MainActivity : AppCompatActivity() {  
    private val viewModel: MainViewModel by viewModels()  
  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContentView(R.layout.activity_main)    
    }  
}
```

The `MainViewModel` looks like this:

```kotlin
class MainViewModel : ViewModel() {  
    private val repository = Repository()  
  
    val state: LiveData<Int> = repository.state.asLiveData()  
}
```

We have a reference to a repository that will act as our trivial data layer. The `ViewModel` also exposes a state as a `LiveData` object converted from a `StateFlow` kept inside the repository using the `asLiveData()` function mentioned before.

Here is what the repository looks like:

```kotlin
class Repository {  
    private val _state = MutableStateFlow(-1)  
    val state: StateFlow<Int> = _state  
  
    suspend fun update() {  
        _state.emit(Random.nextInt(until = 1000))  
    }  
}
```

It's a simple class with a `StateFlow` wrapping an integer (that starts with an initial value of -1) that also has a method allowing clients to update the state with a new random number between 0 and 1000.

Let's imagine we want to schedule the update as soon as our activity is created. We could do this by creating a method inside the `MainViewModel` called `init()` that we would call inside the `onCreate()` of our activity:

```kotlin
// MainViewModel
fun init() {
    // update() is suspending, so we launch a new coroutine here
    viewModelScope.launch {  
        repository.update()
    }  
}

// MainActivity
override fun onCreate(savedInstanceState: Bundle?) {  
    super.onCreate(savedInstanceState)  
    setContentView(R.layout.activity_main)  
  
    viewModel.init()
}
```

Now, during the creation of our activity, a new coroutine will be launched that will eventually call `update()` on the repository, generating a random number and emitting it to the state.

Additionally, let's suppose that there is a requirement to send an analytical event containing the newly generated number in our `ViewModel`. We could write a `sendAnalyticalEvent()` method in our `ViewModel` that we will run right after calling the `update()` method on the repository:

```kotlin
// MainViewModel
fun init() {  
    viewModelScope.launch {  
        repository.update()  
        sendAnalyticalEvent() // <-- NEW
    }  
}  
  
private fun sendAnalyticalEvent() {  
    // Typically, we would schedule a network request here  
  
    val liveDataValue = state.value  
    val flowValue = repository.state.value  
    Log.d("Current number in LiveData", "$liveDataValue")  
    Log.d("Current number in StateFlow", "$flowValue")  
}
```

Inside this method, we would typically schedule a network request to our backend servers but in this case, let's see both values from the  `LiveData` and `Flow` in the Logcat instead:

{{< figure 
align=center
width=350
src="/flowtolivedata/1.png" 
>}}

Now, that is quite unexpected. You could argue that the `LiveData` doesn't have the newest value because there wasn't enough time to collect it from the upstream flow, and you might be right. But in this case, not only does the `LiveData` contain an incorrect value, but it's null! And remember, the initial value of the state in the repository was -1. It can only mean one thing - our  `LiveData` didn't collect anything from the `StateFlow`.

The reason is that we haven't started observing the `LiveData` anywhere. Therefore, it's considered **inactive**. And, according to the documentation of the `asLiveData()` function, in this state, `LiveData` won't be collecting any values from the upstream flow:

>Creates a `LiveData` that has values collected from the origin `Flow`.
> 
>The upstream flow collection starts when the returned `LiveData` becomes active (`LiveData.onActive`). If the `LiveData` becomes inactive (`LiveData.onInactive`) while the flow has not completed, the flow collection will be cancelled after `timeoutInMs` milliseconds unless the `LiveData` becomes active again before that timeout (to gracefully handle cases like Activity rotation).

Once we start observing the state in our activity (hence making the `LiveData` active), it will contain the correct (newest) value:

```kotlin
// MainActivity
override fun onCreate(savedInstanceState: Bundle?) {  
    super.onCreate(savedInstanceState)  
    setContentView(R.layout.activity_main)  
  
    viewModel.init()  
    viewModel.state.observe(this) { // <-- NEW  
        Log.d("Current number in MainActivity", "$it")  
    }  
}
```

And this is the output from the Logcat:

{{< figure 
align=center
width=350
src="/flowtolivedata/2.png" 
>}}

In this example, we use `StateFlow`, but the same rules apply to `SharedFlow`. Furthermore, it would be even worse because any events sent to a `SharedFlow` while the `LiveData` is inactive would be permanently lost (`SharedFlow`, by default, doesn't replay any values to new subscribers).

## Summary

Keep in mind that `LiveData` converted from a `Flow` using the `asLiveData()` function will behave slightly differently than expected. It will emit data only when there are active observers. To me, this behavior makes sense because we generally wouldn't be interested in `LiveData` values if we didn't observe it anywhere.

Regardless, there might be some use cases when you want to access the current value of your `LiveData` in your `ViewModel` before you start observing it. After reading this post, I hope you won't be surprised when encountering seemingly incorrect values in such situations.