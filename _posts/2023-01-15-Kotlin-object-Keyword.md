---
title: How To Use the Kotlin `object`
layout: post
date: 23-01-15
tags:
  - kotlin
  - singleton
---

The `object` keyword is used to declare a singleton, object expressions (which are analogous to Java's anonymous inner classes), and companion objects, among other uses. Here’s a rundown of how it's employed:

### 1. Singleton

In Kotlin, the `object` keyword is used to declare a singleton instance directly. Unlike Java, where implementing a singleton requires a private constructor and a static method to obtain the instance, Kotlin simplifies this process. When we declare an `object`, Kotlin ensures that exactly one instance of this class is created, and this instance is globally accessible throughout our application.

```kotlin
object MySingleton {
    var count: Int = 0
    
    fun doSomething(): Unit {
        println("Doing something with count $count")
    }
}
```

To use it, we can directly call `MySingleton.doSomething()`, and it manages the instance internally.

### 2. Object Expressions

Kotlin uses object expressions to handle cases where we need to create an instance of an anonymous class for immediate use, often as a way to customize or extend existing classes or interfaces without explicitly declaring a new subclass.

```kotlin
val listener = object : MyInterface {
    override fun onEvent() {
        println("Event occurred")
    }
}
```

This is particularly useful for event listeners or implementing interfaces for one-off uses.

### 3. Companion Objects

In Kotlin, static members are handled differently than in Java. Kotlin doesn’t have a `static` keyword. Instead, to attach functions or properties to a class rather than instances of it, we can use a companion object. A companion object is declared within a class, and its members can be accessed directly through the class name, simulating Java's static methods or fields.

```kotlin
class MyClass {
    companion object {
        fun factoryMethod() = MyClass()
    }
}
```

We can access `factoryMethod()` directly via `MyClass.factoryMethod()` without needing an instance of `MyClass`.
