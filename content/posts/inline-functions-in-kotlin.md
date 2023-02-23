---
title: Inline functions in Kotlin
date: 2023-02-23
summary: A comprehensive guide to inline functions, covering their purpose, practical examples, and tips for when to use or avoid them.
---

## Introduction

In this post, I would like to explain in detail what inline functions are and what problems they address. Additionally, I will present some practical examples and tips on when it's appropriate to use inline functions and when to avoid them.

Before we go any further, let's first understand what inline functions do. The general idea is simple. If you mark a function with the `inline` modifier, the compiler will copy the body of that function and paste it at every call site.

As a result, this code:

```kotlin
inline fun foo() {
    print("Body of the foo() function")
}

fun main() {
    foo()
}
```

after compilation will be ultimately equivalent to this:

```kotlin
inline fun foo() {
    print("Body of the foo() function")
}

fun main() {
    print("Body of the foo() function")
}
```

What happened here is instead of calling `foo()`, the compiler copied its content and pasted it into the body of the `main()` function.

Now that we have a basic intuition for inline functions, we can move on to discuss their applications.
 
## Overhead of lambdas and anonymous functions

In Kotlin, functions are [first-class citizens](https://en.wikipedia.org/wiki/First-class_function). They can be stored in variables or data structures, used as arguments, or returned from other functions. However, this feature has an important implication. Functions in these scenarios are represented as objects, which impose certain runtime penalties.

Let's illustrate this with an example. Suppose we write a function called `takeUntil` (which is surprisingly not in the standard library) that would take all the elements from an iterable until a provided predicate is satisfied.

This is what it could look like:

```kotlin
fun <T> Iterable<T>.takeUntil(predicate: (T) -> Boolean): List<T> {  
    val list = ArrayList<T>()  
    for (item in this) {  
        list.add(item)  
        if (predicate(item))  
            break  
    }  
    return list  
}
```

We define it as an extension function and specify one argument, `predicate`, which is a function that will control when we stop taking elements.

> In Kotlin, functions that take other functions as parameters, or return a function, are called High-order functions.

Let's see what the decompiled bytecode for this function looks like in Java:

> In IntelliJ (or Android Studio), you can get the decompiled bytecode by going to `Tools` > `Kotlin` > `Show Kotlin Bytecode` and then clicking `Decompile` button to get the Java code.

```java
public static final List takeUntil(@NotNull Iterable $this$takeUntil, @NotNull Function1 predicate)
```

Notice that our `predicate` argument has a special `Function1` type (this `1` in `Function1` corresponds to the number of arguments our lambda accepts), which confirms that lambdas are indeed represented as objects. We must provide an instance of the `Function1` interface every time we call the `takeUntil` function.

Now, let's write some code to test our new function and see how lambdas that we pass to the `takeUntil` function are converted to objects of the `Function1` type under the hood:

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7)  
    println(numbers.takeUntil { it >= 5 })
}
```

When executed, this code prints a correct result (a list of numbers from 1 to 5 included), but we're interested in something else here. Let's see the decompiled version of the code above:

```java
List numbers = CollectionsKt.listOf(new Integer[]{1, 2, 3, 4, 5, 6, 7});  
List var1 = takeUntil((Iterable)numbers, (Function1)null.INSTANCE);
```

As you can see, the `takeUntil` function takes a mysterious `(Function1)null.INSTANCE` object as an argument. Where does it come from? 

Unfortunately, the generated `Function1` interface doesn't reflect in the decompiled Java code, so we must dive deeper - into the bytecode itself. 

I will skip all the irrelevant code for our discussion and present only the interesting parts.

First, we can see a class created that implements the `Function1` interface.

```java
final class MainKt$main$1 extends kotlin/jvm/internal/Lambda implements kotlin/jvm/functions/Function1
```

The bytecode for that class is equivalent to the lambda body that we've passed to the `takeUntil` function.

Our `main()` function is represented like this:

```kotlin
GETSTATIC MainKt$main$1.INSTANCE : LMainKt$main$1;
CHECKCAST kotlin/jvm/functions/Function1
INVOKESTATIC MainKt.takeUntil (Ljava/lang/Iterable;Lkotlin/jvm/functions/Function1;)Ljava/util/List;
```

The `GETSTATIC` [instruction](https://en.wikipedia.org/wiki/List_of_Java_bytecode_instructions) gets a class's static field value, which means we get a singleton here that is ultimately passed to the `takeUntil` function.

Let's do a quick experiment to confirm that if we put our code in a loop that runs 1000 times, we still use that singleton, and we don't create 1000 new objects:

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7)  
repeat(1000) {  
    println(numbers.takeUntil { it >= 5 })  
}
```

After getting the bytecode, we can confirm that the `GETSTATIC` instruction is still used, so no new objects are being created.

```kotlin
GETSTATIC MainKt$main$1$1.INSTANCE : LMainKt$main$1$1;
CHECKCAST kotlin/jvm/functions/Function1
INVOKESTATIC MainKt.takeUntil (Ljava/lang/Iterable;Lkotlin/jvm/functions/Function1;)Ljava/util/List;
```

So far, nothing dramatic. The bytecode contains an additional class implementing the `Function1` interface that represents our lambda passed to the `takeUntil` function. Additionally, a singleton was used, so there is only one instance of that class at runtime.

However, the situation is **entirely different** when we start capturing variables in lambdas.

Let's store the value we had in the lambda in a variable called `limit` and use it instead:

```kotlin
val limit = 5
val numbers = listOf(1, 2, 3, 4, 5, 6, 7)
repeat(1000) {
    println(numbers.takeUntil { it >= limit })
}
```

And here's the bytecode representation:

```kotlin
INVOKESPECIAL MainKt$main$1$1.<init> (I)V
CHECKCAST kotlin/jvm/functions/Function1
INVOKESTATIC MainKt.takeUntil (Ljava/lang/Iterable;Lkotlin/jvm/functions/Function1;)Ljava/util/List;
```

As you can see, there is no `GETSTATIC` instruction anymore. Instead, there is `INVOKESPECIAL` that calls `<init>`. This line means we are creating a new object of that class. Given that we run this code 1000 times, the same number of new objects will be created at this point.

It's worth remembering that when we create lambdas that capture closures, there will **always** be a new object created, as the above experiment proved. However, in most cases, that won't be a huge problem (unless you deal with a performance-critical application).

Regardless, the solution is to use inline functions. When we mark our `takeUntil` function as `inline`, this is the decompiled Java code that we get for the `main()` function:

```java
List numbers = CollectionsKt.listOf(new Integer[]{1, 2, 3, 4, 5, 6, 7});  
int limit = 5;  
short var2 = 1000;  
  
for(int var3 = 0; var3 < var2; ++var3) {  
   int var5 = false;  
   Iterable $this$takeUntil$iv = (Iterable)numbers;  
   int $i$f$takeUntil = false;  
   ArrayList list$iv = new ArrayList();  
   Iterator var9 = $this$takeUntil$iv.iterator();  
  
   while(var9.hasNext()) {  
      Object item$iv = var9.next();  
      list$iv.add(item$iv);  
      int it = ((Number)item$iv).intValue();  
      int var12 = false;  
      if (it >= limit) {  
         break;  
      }   
    }
    
   List var13 = (List)list$iv;  
   System.out.println(var13);  
}
```

The content of the `takeUntil` function has been copied (inlined) here. Our lambda is no longer represented as an object, so we avoid an additional overhead. Instead, it's been inlined into the if statement (`if (it >= limit)`).

## Better control flow

Another benefit of using inline functions is introducing a better control flow. 

Let's consider a function that is supposed to find a user with a given id or throw an error when there is no such user:

```kotlin
fun getUser(users: List<User>, id: String): User {  
    users.forEach { user ->  
        print("Inspecting user: $user")  
        if (user.id == id) {  
            return user  
        }  
    }  
    error("User with id $id not found.")  
}
```

We iterate over the list of users and check if the current user is the one we seek. If it is, we immediately return that user from our `getUser` function. If we reach the end of the list, we throw an error.

Notice that we use a `return` statement not inside the `getUsers` function but in a lambda passed to the `forEach`. This is only possible because the `forEach` function has been marked with the `inline` modifier.

> Such returns (located in a lambda, but exiting the enclosing function) are called _non-local_ returns.

If we write our own version of `forEach` that is not inlined and use it instead, returning from a lambda won't be possible:

```kotlin
// This is the same function as forEach, but without the inline modifier
fun <T> Iterable<T>.noInlineForEach(action: (T) -> Unit) {  
    for (element in this) action(element)  
}
```

```kotlin
fun getUser(users: List<User>, id: String): User {  
    users.noInlineForEach { user ->  
        print("Inspecting user: $user")  
        if (user.id == id) {  
            return user  
        }  
    }  
    error("User with id $id not found.")  
}
```

We are getting an error that says "'return' is not allowed here".

The reason why returning with the regular `forEach` is possible is that when a lambda is inlined, it's clear that the `return` statement applies to the top-level function (`getUsers`) because there are no other function calls made or objects created.

## Noinline

If you don't want all of the lambdas passed to an inline function to be inlined, you can use the `noinline` modifier.

Consider the following inline function `execute` that decides which API to call based on whether the user is part of an experiment or not:

```kotlin
inline fun execute(  
    isPartOfExperiment: (User) -> Boolean,  
    callOldApi: () -> Unit,  
    callNewApi: () -> Unit  
) {  
    // ...
    val callApi = if(isPartOfExperiment(user)) callNewApi else callOldApi  
    // ...
}
```

We want to store the correct lambda in a variable called `callApi` and call it later at some point, but that's not possible. We get the "Illegal usage of inline-parameter" error. We can't store inlined lambdas in variables (or pass them to other functions) because they aren't objects anymore. 

To solve this, we can add `noinline` modifiers to two of our lambdas, and our code compiles successfully:

```kotlin
inline fun execute(  
    isPartOfExperiment: (User) -> Boolean,  
    noinline callOldApi: () -> Unit,  
    noinline callNewApi: () -> Unit  
) {  
    // ...
    val callApi = if(isPartOfExperiment(user)) callNewApi else callOldApi  
    // ...
}
```

## Crossinline

In some cases, we still want to get all the benefits of using inline functions but without giving the possibility to use a `return` statement inside lambdas passed as parameters. 

To indicate that the lambda parameter of the inline function cannot use non-local returns, we can mark it with the `crossinline` modifier.

Take a look at our little helper `benchmark` function that allows us to measure the execution time of a lambda passed as a parameter:

```kotlin
inline fun benchmark(run: () -> Unit): Long {  
    val duration = measureTime {  
        run()  
    }
    return duration.inWholeMilliseconds  
}
```

Now, for a little experiment, let's try to run the following code:

```kotlin
fun main() {
    val time = benchmark { return }  
    print(time)
}
```

It compiles successfully because the `benchmark` function is marked with the `inline` modifier, and using non-local returns is possible in such cases.

If we run it, the console won't print anything. That's because the `return` statement returns not from the `benchmark` function but immediately from the `main()`, thus ending the program.

If we don't want to allow code like this, we can use the `crossinline` modifier, and the `main()` function won't even compile. We will get the "'return' is not allowed here" error:

```kotlin
inline fun benchmark(crossinline run: () -> Unit): Long {  
    val duration = measureTime {  
        run()  
    }  
    return duration.inWholeMilliseconds  
}
```

## Reified types

Imagine we want to write a function that will return the first item of a given type from a list. For example, we have a `listOf(1, 2, "string", 3)` and would like to get the first string from that list.

Here's one approach to writing such a function:

```kotlin
fun <T> List<*>.firstIsInstance(clazz: Class<T>): T? {  
    @Suppress("UNCHECKED_CAST")  
    return firstOrNull { clazz.isInstance(it) } as T?  
}
```  

And that's what the code to use this function could look like:

```kotlin
val items = listOf(1, 2, "string", 3)
println(items.firstIsInstance(String::class.java))  
```

This code works, but it’s not pretty. We have to pass that not-so-beautiful `String::class.java` to the function. 

Additionally, inside, we have to use a suppression mechanism and call the parameter `clazz`, because `class` is a reserved keyword. A lot of compromises. 

As an alternative, we can rewrite the `firstIsInstance` function as an inline version and utilize *reified type parameters*:

```kotlin
inline fun <reified T> List<*>.firstIsInstance(): T? {  
    return firstOrNull { it is T } as T?  
}
```

Marking the type parameter with the `reified` modifier makes it accessible inside the function. Also, because the function is inlined, we can use normal operators like `as` or `is`, without relying on reflection.

With that change, the client code looks much more elegant:

```kotlin
val items = listOf(1, 2, "string", 3)   
println(items.firstIsInstance<String>())
```

## Avoiding inline functions

So far, we've discussed the benefits of using inline functions. However, it's also worth mentioning when it would be better to avoid them.

First, it doesn't make much sense to add the `inline` modifier to a function that doesn't have any lambdas as parameters because the performance gains are unlikely. The JVM already optimizes small functions and inline them automatically.

Furthermore, you can't use inline functions when hiding implementation details (public inline functions can't call private functions):

```kotlin
inline fun doSomething() {  
    doSomethingPrivately() // Won't compile.
}  
  
private fun doSomethingPrivately() {  
    print("Private function")  
}
```

After learning about the benefits of inline functions, it might be tempting to use them everywhere now but resist the temptation. 

For example, inlining a large function could dramatically increase the size of the generated bytecode because it's copied to every call site. For similar reasons, it's better to avoid inlining recursive calls.

## Summary

This was quite a long post, but I hope you learned a lot from it. As much as I love the [Kotlin documentation](https://kotlinlang.org/docs/home.html), I think the section dedicated to inline functions could use some work. That was the main motivation behind the decision to cover inline functions extensively here, including some practical examples.

If you have any questions, feel free to leave a comment here or reach out to me on [Twitter](https://twitter.com/ClouddJR/).