---
title: How To Design an MVI Project For Android Part 7 - Using Dependency Injection
layout: post
date: 23-09-14
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
  - dependency_injection
  - state_pattern
---

## Hilt

For injecting `Context` into our `ViewModel`, the most appropriate and modern approach in Android development, especially within the scope of recommended practices by Google, is to use **Hilt** for dependency injection. Hilt simplifies the process, makes it so that safe handling of the `Context`, and keeps our code clean and manageable. Here's why Hilt is particularly suited for this task and how to do it:

## Why Hilt?

- **Lifecycle Awareness**: Hilt is designed to be aware of Android lifecycles, ensuring that the correct type of `Context` (application, activity, etc.) is provided where needed, reducing the risk of memory leaks.
- **Simplicity**: Hilt abstracts away much of the boilerplate code needed for dependency injection, making our codebase simpler and more readable.
- **Scalability**: As our project grows, Hilt makes it easier to manage dependencies across different components of our Android app.
- **Integration with Jetpack**: Hilt works seamlessly with other Jetpack components, including `ViewModels`, and follows best practices set by the Android team.

## Add Hilt to our Project

Ensure you have Hilt set up in our project by adding the necessary dependencies and plugins. In our `build.gradle` (project level), you need to include the Hilt Gradle plugin:

```groovy
plugins { 
	...
	id("com.google.dagger.hilt.android") version "2.44" apply false
}

buildscript {  
	dependencies {  
		classpath("com.google.dagger:hilt-android-gradle-plugin:2.44")  
	}  
}
```

And in our `build.gradle` (app/module level), apply the plugin and add Hilt dependencies:

```groovy
plugins {  
	id("kotlin-kapt")  
	id("dagger.hilt.android.plugin")  
}

dependencies {
	implementation("com.google.dagger:hilt-android:2.44")  
	kapt("com.google.dagger:hilt-android-compiler:2.44")
}
```

## The Application Class

In Android development, the `Application` class in Android serves as a base class for maintaining global application state before any activity, service, or receiver objects (components) are created. It's like the entry point to our application, but not exactly the "main" as you would find in other types of programs, like a Java console application or C++ program. Instead, Android decides which component (Activity, Service, etc.) to launch based on the intent it receives and the app's manifest declarations.

### Why You Might Not See an Application Class

It's perfectly normal for small or simple Android projects not to have a custom `Application` class if they don't need to initialize global state or libraries at the application start. our `MainActivity` can indeed seem like the "main" part of our app since it's often the entry point for user interaction.

### Purpose of an Application Class

However, when you start needing to initialize global libraries, handle application-wide state, or work with frameworks that require initialization (like Hilt for dependency injection), defining a custom `Application` class becomes necessary. It allows you to:

- Initialize libraries at the start of our app's lifecycle, before any activity is started.
- Maintain global application state or resources.
- Use services like dependency injection frameworks more effectively.

### How to Create and Use an Application Class with Hilt

1. **Create a Custom Application Class**: The `@HiltAndroidApp` annotation is important here; it sets up Hilt for dependency injection across the application. This is all we really need at this point.
    ```kotlin
    import android.app.Application
    import dagger.hilt.android.HiltAndroidApp

    @HiltAndroidApp
    class TableRollerApp : Application() {
        // Initialization code here
    }
    ```


2. **Declare It in our AndroidManifest.xml**: Once you've created our custom `Application` class, you need to declare it in our `AndroidManifest.xml` so the Android system knows to use it:

    ```xml
    <application
        android:name=".TableRollerApp"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name">
        <!-- our activities and other components -->
    </application>
    ```
   
We would replace `".TableRollerApp"` with the actual path to the custom `Application` class if it's not in the root package. In this case, we've put it on the same level as `MainActivity` in our file tree. You can see in the manifest that MainActivity is still in the `LAUNCHER` category.

### Injecting Context into the ViewModel

Use `@HiltViewModel` on our ViewModel and inject `Context` using `@ApplicationContext` to ensure you're using the application context, which is safe:
```kotlin
@HiltViewModel
class LootTableViewModel @Inject constructor(
    @ApplicationContext private val context: Context) 
    : ViewModel() {
    // Use context as needed
}
```

The `@ApplicationContext` is going to get flagged as "This field leaks Context." This warning, when you're using `@ApplicationContext` in a `ViewModel` with Hilt, can be a bit misleading because using `@ApplicationContext` is actually the recommended practice to avoid context leaks. This annotation makes it so that you are provided with the application context, not an activity context, which is safe to retain across the configuration changes since it's tied to the lifecycle of the entire application, not just a single activity or view.
##### Understanding the Context
- **Application Context**: It's a context tied to the lifecycle of the application, not an activity. It's safe to hold onto this context for long periods, or even for the lifetime of our app, which is why it's recommended for use in `ViewModels` and other long-lived components.
- **Activity Context**: It's tied to the lifecycle of an activity. Holding a reference to it in a `ViewModel` or any component that outlives an activity instance can lead to memory leaks.

### Prepare our Activity or Fragment

Ensure our `Activity` or `Fragment` is ready to work with Hilt by annotating them with `@AndroidEntryPoint`:

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    // Now Hilt can inject dependencies into MainActivity, and you can obtain ViewModels via Hilt
}
```

This setup makes it so that our ViewModel receives the `Context` in a safe, efficient, and scalable manner, adhering to best practices recommended by the Android team.
