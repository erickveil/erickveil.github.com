---
title: How To Use the Kotlin `when` Statement
layout: post
date: 23-02-07
tags:
  - kotlin
---
The `when` statement simplifies branching logic, much like the switch-case statement in languages like Java or C++. 

1. **Basic Usage**: At its simplest, `when` checks a variable against various values or conditions and executes the block of code corresponding to the first matching condition.

   ```kotlin
   when (x) {
       1 -> print("x == 1")
       2 -> print("x == 2")
       else -> { // Note the block
           print("x is neither 1 nor 2")
       }
   }
   ```

2. **Checking Against Multiple Values**: We can check a value against multiple possible matches in a single line.

   ```kotlin
   when (x) {
       0, 1 -> print("x is 0 or 1")
       else -> print("otherwise")
   }
   ```

3. **Using Arbitrary Conditions**: Instead of checking for equality, we can evaluate more complex conditions.

   ```kotlin
   when {
       x < 10 -> print("x is less than 10")
       x > 10 -> print("x is greater than 10")
       else -> print("x is exactly 10")
   }
   ```

4. **Returning Values**: `when` can also be used as an expression that returns a value. This is particularly useful in assignments or return statements.

   ```kotlin
   val result = when (x) {
       1 -> "x is one"
       2 -> "x is two"
       else -> "x is unknown"
   }
   println(result)
   ```

5. **Checking Types**: we can use `when` to check the type of an object by using the `is` keyword. This can simplify type checks and casting.

   ```kotlin
   val response = when (val msg = fetchMessage()) {
       is String -> "Message is a string"
       is Int -> "Message is an integer"
       else -> "Unknown message type"
   }
   ```

6. **Without an Argument**: If we omit the argument, `when` works like a series of `if-else if` statements. This is what makes it possible to use arbitrary conditions.

   ```kotlin
   when {
       x.isOdd() -> print("x is odd")
       x.isEven() -> print("x is even")
       else -> print("x is funny")
   }
   ```


# Sealed Classes

A `sealed` class is a type of class that restricts class hierarchies. A `sealed` class cannot be instantiated on its own and can only be subclassed within the file in which it is declared. This allows us to define a closed set of subclasses that are known, making it easier to handle them exhaustively when we're using them in, for example, `when` expressions. 

It solves the problem: Is there another subclass/state/strategy hidden somewhere else in this giant code-base that I'm not accounting for?

This is useful in cases where a function or method can only return a limited, fixed number of types, such as in scenarios where we are using a [state](https://refactoring.guru/design-patterns/state) or [strategy](https://refactoring.guru/design-patterns/strategy) pattern.


```kotlin
sealed class Result
data class Success(val message: String) : Result()
data class Failure(val error: String) : Result()
data class Loading() : Result()

fun handleResult(result: Result) {
    when (result) {
        is Success -> println(result.message)
        is Failure -> println(result.error)
        is Loading -> println("Loading...")
        // No `else` branch needed as all cases are covered
    }
}
```

In this example, `Result` is a sealed class with three possible types: `Success`, `Failure`, and `Loading`. When using a `when` expression to handle instances of `Result`, we are guaranteed to handle all possible cases. Because all subclasses must be declared in the same file as the sealed class, this tightly couples the subclasses with their parent, ensuring that the set of subclasses is always known and closed.

When we're classing a type for state or strategy patterns, the classes usually aren't very large (If they are, we might want to refactor). So having all the classes in one file for all subclasses should not create an unmaintainable mega-file.
