---
title: How To Use Kotlin Coroutines
layout: post
date: 23-04-19
tags:
  - kotlin
  - android
  - coroutines
  - async
---
## A Non-Blocking Example

This example will simulate fetching data from a remote service in the background and then updating the UI (or in this case, printing to the console, as we're working within a text-based example) without blocking the main thread.

For this example, I'll include a non-blocking suspending function that simulates fetching data from a remote service with a delay (to mimic network latency) and then continues with updating the UI on the main thread after the data is fetched.

```kotlin
import kotlinx.coroutines.*

fun main() {
    // Create a CoroutineScope that runs on the main thread
    val mainScope = CoroutineScope(Dispatchers.Main)

    mainScope.launch {
        println("Fetching data from the network...")
        val data = fetchDataFromNetwork() // This will not block the main thread
        println("Data fetched: $data")

        // Update UI or process data here (still on the main thread)
        println("Updating UI with the fetched data.")
    }

    // Assume Dispatchers.Main is simulated for the purpose of this example.
    // In a real application, this would typically be our Android Activity or Fragment.
    println("This will print while data is being fetched.")
    Thread.sleep(3000) // Simulating some other work
}

// Simulate a network call
suspend fun fetchDataFromNetwork(): String {
    withContext(Dispatchers.IO) { // Switch to a background thread
        delay(1000) // Simulate network delay
        return@withContext "Sample Data" // Return data
    }
}
```

- **Non-Blocking Network Call**: The `fetchDataFromNetwork` suspending function simulates a network call on a background thread using `withContext(Dispatchers.IO)`. That way we can make sure the main thread isn't blocked by the network call.

- **Main-Safe**: After fetching the data, the coroutine continues execution on the `Dispatchers.Main` context, allowing safe updates to the UI.

- **Structured Concurrency**: The coroutine is launched within a `CoroutineScope` tied to the main thread (`Dispatchers.Main`). 

- **Simultaneous Execution**: The `println` statement after launching the coroutine shows that the program continues execution without waiting for the coroutine to complete. 

## Use in MVI

In an MVI project I was working on, I used the following function to load a JSON file and parse it into a Data Model. 
```kotlin
private fun loadLootTable() { 
	viewModelScope.launch { 
		lootTable = repository.getLootTable() 
	} 
} 
```

By using a coroutine to load the file, I make sure that my UI remains responsive while the data is being loaded. 

Here, we use a different scope than the main dispatcher, **`viewModelScope`**. This is a CoroutineScope tied to the ViewModel lifecycle. Using this scope lets me make sure that the coroutine is canceled when the ViewModel is cleared, preventing any potential memory leaks or unnecessary work if the ViewModel is no longer in use. 

## A Blocking Example

Imagine we have a simple user interface with a text view or label that we want to update with data fetched from a network. We'll simulate the network request with a suspending function that delays for a short period to mimic the latency of a real network call.

```kotlin
import kotlinx.coroutines.*
import kotlin.random.Random

// Simulate a network request that fetches user data
suspend fun fetchUserData(): String {
    delay(2000) // Simulate network delay
    return "User ${Random.nextInt(1000)}" // Return a mock user name
}

fun main() = runBlocking {
    val job = launch(Dispatchers.Main) { // Launch a coroutine in the UI thread
        val userData = fetchUserData() // Fetch user data (this suspends the coroutine)
        println("Fetched user data: $userData") // Update UI with fetched data
    }
    job.join() // Wait for the coroutine to finish
}
```

In this example:

- We define a `suspend` function `fetchUserData` that mimics fetching data from a network. It uses `delay` to simulate network latency and then returns a mock user name.
- In the `main` function, which acts as our entry point, we use `runBlocking` to create a coroutine scope. `runBlocking` is usually used in main functions and tests to wait for a coroutine to complete.
- Inside `runBlocking`, we use the `launch` coroutine builder to start a new coroutine. We specify `Dispatchers.Main` as the coroutine context. Note: In real applications, `Dispatchers.Main` is available in environments with a UI main thread, such as Android or JavaFX applications. In a console application, we'd use `Dispatchers.Default` or another appropriate dispatcher.
- The coroutine calls `fetchUserData` and suspends until the data is fetched without blocking the main thread. After receiving the data, it prints the fetched user data to simulate updating the UI.
- Finally, `job.join()` waits for the coroutine to finish its execution. In a real application, especially in GUI applications like Android, we wouldn't typically join the coroutine like this, as the coroutine lifecycle would be managed by the application's architecture components (e.g., a ViewModel).

`join()` is useful when we need to wait for a task to complete before proceeding. This is common in structured concurrency where the outcome of a coroutine impacts subsequent actions.

In UI applications, direct use of `join()` is less common because it can lead to blocking the main thread, especially if not used carefully with appropriate dispatchers or within a non-blocking context.

# Suspend

The `suspend` keyword in Kotlin allows asynchronous and non-blocking programming. It marks a function as a suspending function, which means that the function can suspend the execution of a coroutine without blocking the thread on which it's running. This capability is foundational for writing asynchronous code in Kotlin that's both efficient and easy to manage.

### Characteristics of Suspended Functions:

- **Non-Blocking**: When a suspending function is called, it doesn't block the thread it's running on. Instead, it suspends the coroutine in which it's called. This is crucial for performing long-running operations, such as network requests or database transactions, without freezing the application's user interface.

- **Can Be Called from Coroutines or Other Suspending Functions**: A suspending function can only be called from within another suspending function or a coroutine builder block (like `launch` or `async`). 

- **Cooperative**: Suspending functions are cooperative. They suspend the execution at specific points, which are defined by either calling other suspending functions or explicitly suspending the coroutine using functions like `delay()`.

### How It Works:

Under the hood, Kotlin transforms suspending functions into regular functions that take an additional parameter: a continuation. Continuations are part of Kotlin's coroutines library, and they manage the state of the coroutine, including its suspension and resumption. This transformation is handled by the Kotlin compiler, which allows developers to write asynchronous code that looks synchronous, improving readability and maintainability.

### Example:

Here's a simple example of a suspending function that delays for one second:

```kotlin
suspend fun doSomething() {
    delay(1000)  // Suspends the coroutine for 1 second without blocking the thread
    println("Done!")
}
```

This function can be called from within a coroutine or another suspending function like so:

```kotlin
fun main() = runBlocking {  // Creates a coroutine scope
    launch {  // Launches a new coroutine
        doSomething()  // Calls the suspending function
    }
}
```

# Specifying a Coroutine Context or Dispatcher

### Main vs Background Thread:
- Use `Dispatchers.Main` when we need to update the UI after performing a task. This is crucial in Android development, where UI changes must be made on the main thread.
- Use `Dispatchers.IO` for I/O operations like reading from or writing to files, databases, or network calls.
- Use `Dispatchers.Default` for CPU-intensive tasks that can be run concurrently.

### Environment:
- In non-Android applications (like server-side or Kotlin/JVM applications), `Dispatchers.Main` might not be relevant, and we’d default to `Dispatchers.Default` or `Dispatchers.IO` based on the task type.
- In Android, `Dispatchers.Main` is often used within `viewModelScope.launch` for tasks that result in UI updates.




