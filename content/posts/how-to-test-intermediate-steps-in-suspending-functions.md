---
title: How to test intermediate steps in suspending functions
date: 2022-09-30
summary: Testing the final result of a suspending function is easy, but what about verifying what happens inside it during the execution?
---

## Introduction

Let's look at a simple suspending function that fetches some data from a server. Before executing a network call, it sets the `isLoading` variable to `true`. Then, after finishing successfully, it sets it back to `false` and returns the loaded data:

```kotlin
class Repository {  
    var isLoading = false  
  
    suspend fun fetchData(): String {  
        isLoading = true  
        delay(500) // Simulate a network call  
        isLoading = false  
        return "Loaded data"  
    }  
}
```

For simplicity, let's ignore all the additional things we would normally take care of in a real project, like error handling, making the actual network call, parsing the result, etc. Instead, let's focus on the aspect of testing this snippet of code.

Testing the final result of the `fetchData` function is relatively easy with the help of some tools from the `kotlinx.coroutines.test` library. This is what such a test could look like:

```kotlin
@Test  
fun `repository should return loaded data`() = runTest {  
    // given  
    val repository = Repository()  
  
    // when  
    val result = repository.fetchData()  
  
    // then  
    assertEquals("Loaded data", result)  
}
```

> Notice how this test doesn't look much different than what we would write if we were testing a regular function.

`runTest` is an extremely convenient function. It behaves similarly to `runBlocking`, with the difference that the code that it runs will skip delays. Thanks to this, we don't have to wait 500 ms (because of the `delay(500)` in our `fetchData` function) for the test to finish.

The code above works just fine, but what if we want to have more control over this test? We would like to make sure that the `fetchData` function indeed sets correct values to the `isLoading` variable - before and after making a network call.

## Solution

We can take advantage of the fact that `runTest` executes our test body in a new coroutine launched on a `TestScope`. 

This scope uses a special kind of dispatcher - `StandardTestDispatcher`. Usually, dispatchers are used to control the thread on which our coroutines should run. This dispatcher does a bit more - it supports delay-skipping using a scheduler that operates on virtual time (`TestCoroutineScheduler`). 

Coroutines started with the `StandardTestDispatcher` won't run immediately. Instead, the dispatcher always sends them to its scheduler. Then, it's our job to trigger their execution by controlling the virtual time. For this, we can use functions available on the `TestCoroutineScheduler`:  `advanceTimeBy`, `runCurrent` or `advanceUntilIdle`.

To see this in action, look at the code below:

```kotlin
@Test  
fun `child coroutine`() = runTest {  
    // 1  
    launch {  
        // 3  
    }  
    // 2  
    advanceUntilIdle() // Run all the scheduled tasks
    // 4  
}
```

When we call `launch`, the coroutine is not executed immediately. Instead, as mentioned earlier, it's sent to the scheduler. Only when we call `advanceUntilIdle` (or any other function controlling the virtual time), the child coroutine is executed.

> Notice that we can call `advanceUntilIdle` directly on the `TestScope`. It's because it's defined as an extension function that internally delegates the call to the scheduler. The same is true for the remaining functions modifying the virtual time.

This behavior allows us to call the function we want to test inside a child coroutine. That coroutine will be sent to the scheduler and its execution is fully controllable by us. 

We can advance the virtual time to the point where the data is about to be loaded and verify that the `isLoading` is properly set to `true`. Then we can run all the tasks scheduled at that time (remember the `delay(500)` that we had in our function?) by using `runCurrent`. Lastly, we can assert that the `isLoading` variable is back to `false`:

```kotlin
@Test  
fun `repository should indicate that it's loading data`() = runTest {  
    // given  
    val repository = Repository()  
    launch {  
        repository.fetchData()  
    }  
  
    // when  
    advanceTimeBy(500)  
  
    // then  
    assertTrue(repository.isLoading)  
  
    // when  
    runCurrent()  
  
    // then  
    assertFalse(repository.isLoading)  
}
```

## Regular functions that launch coroutines

Previously, we looked at testing intermediate steps in suspending functions. But in some cases, we want to test a regular, non-suspending function that launches a new coroutine inside. It's very common in many Android projects where you can see a code similar to this one:

```kotlin
class TasksViewModel(private val repository: TasksRepository) : ViewModel() {  
    private val _isLoading = MutableStateFlow(false)  
    val isLoading: StateFlow<Boolean> = _isLoading  
  
    fun getTasks() {  
        viewModelScope.launch {  
            _isLoading.emit(true)  
            repository.getTasks()  
            _isLoading.emit(false)  
        }  
    }  
}
```

We have a `ViewModel` that exposes a function called `getTasks`. Internally, it launches a new coroutine using a `viewModelScope` in which it calls a suspending function `getTasks` from our repository. Besides, it correctly updates the `isLoading` state before and after.

Here's what the `TasksRepository` looks like (including the implementation that will be used in tests):

```kotlin
interface TasksRepository {  
    suspend fun getTasks(): List<Task>  
}  
  
class InMemoryTasksRepository : TasksRepository {  
    override suspend fun getTasks(): List<Task> {  
        delay(500) // Simulate a network call  
        return listOf(  
            Task("1", "Task 1"),  
            Task("2", "Task 2")  
        )  
    }  
}  
  
data class Task(val id: String, val name: String)
```

We can't directly use the same approach as previously, because the coroutine inside `getTasks` is launched on the `viewModelScope`. This scope knows nothing about the `TestScope` that is running our test body. This means that `viewModelScope.launch {...}` would be launched immediately instead of being scheduled by the `StandardTestDispatcher`.

To solve this, we need to force the `viewModelScope` to use the `StandardTestDispatcher` that we could control from the outside.

Luckily, according to the source code, `viewModelScope` is using a `Dispatchers.Main.immediate`. The main dispatcher can easily be replaced during tests using `Dispatchers.setMain`. There, we can pass our `StandardTestDispatcher`. 

Here's how we could set up our test using the `@Before` and `@After` annotations:

```kotlin
private lateinit var testScheduler: TestCoroutineScheduler

@Before  
fun setUp() {  
	testScheduler = TestCoroutineScheduler()  
	Dispatchers.setMain(StandardTestDispatcher(testScheduler))  
}  

@After  
fun tearDown() {  
	Dispatchers.resetMain()  
}  
```

Before each test, we create a `StandardTestDispatcher`, providing our `TestCoroutineScheduler`, and set it as the main dispatcher (that we reset after each test). 

Thanks to this, the `viewModelScope` will be using this dispatcher and all coroutines launched on this scope will be sent to our scheduler. 

The code of our test will be almost identical to the previous one. The only difference is that we don't launch a child coroutine anymore and we control the virtual time by calling methods on the `testScheduler` variable.

Here is the full code of our test, including the setup:

```kotlin
class ViewModelTest {  
    private lateinit var testScheduler: TestCoroutineScheduler  
  
    @Before  
    fun setUp() {  
        testScheduler = TestCoroutineScheduler()  
        Dispatchers.setMain(StandardTestDispatcher(testScheduler))  
    }  
  
    @After  
    fun tearDown() {  
        Dispatchers.resetMain()  
    }  
  
    @Test  
    fun `should indicate loading when getting the data`() {  
        // given  
        val viewModel = TasksViewModel(InMemoryTasksRepository())  
        viewModel.getTasks()  
  
        // when  
        testScheduler.advanceTimeBy(500)  
  
        // then  
        assertTrue(viewModel.isLoading.value)  
  
        // when  
        testScheduler.runCurrent()  
  
        // then  
        assertFalse(viewModel.isLoading.value)  
    }  
}
```

Notice that we don't even have to run our test body inside `runTest` anymore. That's because we are not calling any suspending functions here - `getTasks` is a regular function.

## Conclusion

I hope this post showed you how we can easily make our tests more powerful and controllable when we write code that uses coroutines. 

I first learned the trick of launching a child coroutine in a test to control it from the outside in a book I highly recommend - [Kotlin Coroutines: Deep Dive](https://kt.academy/book/coroutines) by Marcin Moskala.

If you have any questions, feel free to reach me on [Twitter](https://twitter.com/ClouddJR/).