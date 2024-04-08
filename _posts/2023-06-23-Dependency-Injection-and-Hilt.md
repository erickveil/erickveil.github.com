---
title: How To Do Dependency Injection With Hilt
layout: post
date: 23-06-23
tags:
  - android
  - kotlin
  - dependency_injection
---
### What is Dependency Injection?

Dependency Injection is a design pattern that aims at making our code more modular and testable. It involves providing the objects that an object needs (its dependencies) rather than having it construct them itself. 

In the context of Android development, DI helps in managing the instantiation of classes that require context or configuration. 

For example, suppose we have a class that requires a reference to a `SharedPreferences` instance. Instead of instantiating `SharedPreferences` directly within the class, we would inject this dependency. This makes our class easier to test (since we can inject a mock `SharedPreferences` instance) and reduces the class's responsibility to just using `SharedPreferences`, not creating it.

### Enter Hilt

Hilt is a dependency injection library for Android that reduces the boilerplate of doing manual dependency injection in our project. It is built on top of Dagger, using many of its features, but simplifying the implementation with annotations.

### Key Features of Hilt

- **Automated Dependency Injection**: Hilt generates Dagger components and handles their lifecycles automatically, following Android's lifecycle.
- **Easy Configuration**: With annotations, Hilt allows for easier configuration and less boilerplate code than Dagger.
- **Compile-time Validation**: Hilt performs many checks at compile-time, ensuring that our app's dependencies are satisfactorily provided.
- **Scoped Bindings**: Hilt supports defining scopes for dependencies, ensuring that instances live as long as their scope does (e.g., Activity scope, Application scope).

### How to Use Hilt in Android with Kotlin

1. **Add Hilt Dependencies**: First, we need to add Hilt's Gradle plugin and dependencies to our project's build.gradle files.

2. **Annotate our Application Class**: Use the `@HiltAndroidApp` annotation on our Application class to enable Hilt.

3. **Injecting Dependencies**: Use `@Inject` to annotate the constructor of the class where we want a dependency to be injected. Hilt takes care of the rest.

4. **Define Modules and Provide Dependencies**: Sometimes, we need to tell Hilt how to provide instances of a certain type. This is done in a `Module` class annotated with `@Module` and `@InstallIn`, indicating where this module's bindings are available.

5. **Inject into Android Classes**: We can directly inject dependencies into Android classes such as Activities, Fragments, and ViewModels using Hilt annotations.

### Example

```kotlin
// Application class
@HiltAndroidApp
class MyApplication : Application()

// A module that tells Hilt how to provide instances of SomeType
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Singleton
    @Provides
    fun provideSomeType(): SomeType = SomeImplementation()
}

// A class that needs SomeType injected
class MyClass @Inject constructor(private val someType: SomeType) {
    fun doSomething() {
        someType.performAction()
    }
}
```

So we still use the constructor to inject the object as an argument, but the extra Hilt decoration automates a bunch of extra work that we would also have to do, because of how Android scopes things and manages memory and such.

### When to Use Hilt Annotations

- **Large or Complex Projects**: If our project is expected to grow in complexity, using Hilt can save us a lot of headache down the line. It manages our dependency graph automatically, making it easier to handle changes and additions.
- **Android Lifecycle Integration**: For dependencies that need to be tied to Android lifecycle components (like Activities, Fragments, ViewModels, etc.), Hilt provides scoped annotations (e.g., `@ActivityScoped`, `@ViewModelInject`) that handle instantiation and destruction at the appropriate times, reducing memory leaks and other lifecycle-related issues.
- **Shared Dependencies**: If we have dependencies that are used across multiple components or need to maintain a single instance (singleton) across the application, Hilt’s annotations (like `@Singleton` or `@InstallIn`) simplify their configuration and make sure that the framework manages their lifecycle appropriately.
- **Testing and Maintainability**: When our project reaches a certain scale, testing and maintainability become crucial. Hilt’s ability to swap out implementations for testing purposes without changing the actual codebase is a significant advantage.

### When Manual Constructor Injection Might Suffice

- **Small Projects or Prototypes**: For small projects or when rapidly prototyping, the overhead of setting up Hilt might not be worth the benefits. In these cases, passing dependencies directly through constructors is simpler and gets the job done.
- **Limited Dependencies**: If our class only has a few dependencies that are easy to manage or when we're working within a scope not managed by Android (like a pure Kotlin module), manual injection through constructors can be straightforward and efficient.

### Can We Mix Both?

In practice, we might find that a hybrid approach works best. We could use Hilt for the broader application needs, especially for managing shared instances, lifecycle-bound components, and complex dependency graphs, while still manually injecting simpler dependencies or in situations where Hilt's automatic management isn't necessary or beneficial.

It’s also worth noting that adopting Hilt doesn’t mean we have to use it for every single dependency. We can gradually introduce Hilt into an existing project, starting with areas where it provides the most immediate benefits (like ViewModel injection or scoping instances to the lifecycle of Android components) and manually handling simpler cases as needed.

### Injecting ViewModel into Composable Functions

I have a project where I have to provide the ViewModel (`LootTableViewModel`) to the Compose function using Hilt. I've set up the gradle and have my annotations all set and everything works. In the Main Activity, I have to declare the ViewModel this way: 

```kotlin
private val viewModel: LootTableViewModel by viewModels()
``` 

This line doesn't actually instantiate the `LootTableViewModel`. It's creating an injection object that will lazy create the `LootTableViewModel` in a way that Hilt can manage all the things it needs to manage with this injection. It also saves me the boilerplate of bringing in the factory and does all that behind the scenes.

And then provide it to the View this way: 
```kotlin
LootTableUIEnhanced(viewModel) 
```

Notice that we are injecting the `viewModel` into a composable function. This is a common pattern in Jetpack Compose.

In Jetpack Compose, it's typical to pass data objects, including `ViewModels`, as parameters to composable functions. This approach allows the composable functions to observe and react to state changes within these data objects, enabling dynamic UI updates. Here’s how it works and why it’s useful:

1. **Reactivity**: Composable functions can "observe" LiveData or State objects within a ViewModel. When the observed data changes, the composable function automatically recomposes (re-renders), reflecting the latest data. This reactive programming model simplifies the development of dynamic UIs.

2. **Decoupling UI and Data**: By injecting `ViewModels` into composable functions, we decouple our UI structure from our data and business logic. This separation of concerns makes our code more maintainable and testable.

3. **Scoped to Lifecycle**: When we obtain a `ViewModel` in the context of an `Activity` or `Fragment` using `by viewModels()`, it's scoped to the lifecycle of that component. Passing the `ViewModel` to composable functions allows us to maintain this scoping, ensuring that the `ViewModel` survives configuration changes (like screen rotations) and is tied to the lifecycle of the hosting `Activity` or `Fragment`.

### Delegates and the `by` Keyword

The `by` keyword in Kotlin is used to signify delegation, a design pattern where an object delegates one or more of its responsibilities to another object. In Kotlin, this is a language feature known as delegated properties, allowing certain functions of a property—like getting and setting its value—to be handled by another object. This mechanism is versatile and can be used in various contexts beyond just lazy initialization or dependency injection.

### Syntax and Basic Use

The basic syntax for using `by` in the context of delegated properties is:

```kotlin
val/var <property name>: <Type> by <delegate>
```

Here, `<delegate>` is an expression that provides the delegate object, which must have functions `getValue` (and `setValue` for `var` properties) that Kotlin calls whenever the property is accessed or modified.

### Use Cases for Delegation

1. **Lazy Properties**: Probably the most common use of delegation is for lazy initialization of properties using the `lazy` function. The property is only initialized when first accessed, which can save resources and improve performance.

   ```kotlin
   val lazyValue: String by lazy {
       println("computed!") // This block only executes on the first access
       "Hello"
   }
   ```

2. **Observable Properties**: Kotlin's standard library includes delegates like `Delegates.observable()` and `Delegates.vetoable()` for properties that notify listeners of changes.

   ```kotlin
   var name: String by Delegates.observable("<no name>") { prop, old, new ->
       println("$old -> $new")
   }
   ```

3. **Storing Properties in a Map**: This is particularly useful in scenarios like parsing JSON or working with dynamic data. Properties can be delegated to a map, where keys match property names.

   ```kotlin
   class User(val map: Map<String, Any?>) {
       val name: String by map
       val age: Int by map
   }
   ```

4. **Custom Delegates**: We can create our own delegation logic by implementing the `ReadOnlyProperty` or `ReadWriteProperty` interface, depending on whether the property is `val` or `var`. This is useful for custom caching, thread-safety wrappers, or accessing external resources.

   ```kotlin
   class ResourceDelegate<T>(private val loader: () -> T) : ReadOnlyProperty<Any?, T> {
       private var value: T? = null

       override fun getValue(thisRef: Any?, property: KProperty<*>): T {
           if (value == null) {
               value = loader()
           }
           return value!!
       }
   }
   
   fun <T> resourceLoader(loader: () -> T): ReadOnlyProperty<Any?, T> = ResourceDelegate(loader)
   ```

Notice how this closely resembles how I set up singletons in C++. The pattern involves making the default constructor private, providing a static method that returns the instance of the singleton class, and ensuring that the class is only instantiated once.

The custom delegate example in Kotlin can mimic the singleton pattern to some extent by lazily loading and caching a resource, ensuring that it's only created once upon first access. However, the scope and application are a bit different. In the Kotlin example, the focus is on lazy initialization and property delegation, which is a broader concept than just implementing singletons.


# Creating Our Own Hilt Modules

To get the decoupling and testability benefits of Dependency Injection (DI) by injecting an instance of class A into class B, we'll follow a few steps. Assuming we're working in a Kotlin environment and wish to apply these principles perhaps within an Android context or any Kotlin-based application, here's a general approach:

### Step 1: Define Our Classes

First, define our classes. For DI to be most effective, class A should implement an interface, which class B will depend on, rather than depending directly on class A. This interface-based approach enhances testability and decoupling.

```kotlin
interface AInterface {
    fun doSomething()
}

class A : AInterface {
    override fun doSomething() {
        // Implementation details...
    }
}

class B(private val a: AInterface) {
    fun useA() {
        a.doSomething()
    }
}
```

### Step 2: Set Up Dependency Injection

1. **Add Hilt Dependencies**: Make sure our project's `build.gradle` files are set up for Hilt.
2. **Annotate Our Application Class**: Use `@HiltAndroidApp` on our Application class.

   ```kotlin
   @HiltAndroidApp
   class MyApplication : Application()
   ```

3. **Define Modules for Dependency Provisioning**: Create a Hilt module that tells Hilt how to provide instances of `AInterface` and `B`.

   ```kotlin
   @Module
   @InstallIn(SingletonComponent::class)
   object MyModule {
       @Provides
       @Singleton
       fun provideAInterface(): AInterface = A()
   
       @Provides
       fun provideB(aInterface: AInterface): B = B(aInterface)
   }
   ```

4. **Inject Into Our Classes**: With Hilt, injecting into Android framework classes (like Activities, Fragments, etc.) is straightforward. For other classes, we'll usually pass dependencies through their constructors, as shown with class B.

### Step 3: Use Our Classes

With Hilt, once everything is set up, we can inject `B` into our Android components (e.g., Activities, Fragments) and use it:

```kotlin
@AndroidEntryPoint
class MyActivity : AppCompatActivity() {

    @Inject lateinit var b: B

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_my)

        b.useA()
    }
}
```




