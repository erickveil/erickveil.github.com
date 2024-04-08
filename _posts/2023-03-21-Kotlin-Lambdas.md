---
title: How To Use Kotlin Lambdas
layout: post
date: 2023-03-21
tags:
  - closures
  - android
  - kotlin
  - lambdas
---

Here's the general syntax for defining a lambda expression and assigning it to a variable:

```kotlin
val lambdaName: (InputType) -> ReturnType = { input -> 
    // lambda body
}
```

Let's put this into a concrete example. Suppose we have a list of integers and we want to define a lambda that checks if a number is positive. We could write:

```kotlin
val isPositive: (Int) -> Boolean = { number ->
    number > 0
}
```

In this example, `isPositive` is a lambda expression that takes an `Int` as input and returns a `Boolean`, indicating whether the input number is greater than zero. We've explicitly defined the input type `(Int)` and the return type `Boolean` of the lambda.

Once we've defined this lambda, we can use it as an argument to higher-order functions such as `filter`:

```kotlin
val numbers = listOf(-2, -1, 0, 1, 2)
val positiveNumbers = numbers.filter(isPositive)
```

Here, `filter` uses the `isPositive` lambda to filter out only the positive numbers from the list.

If our lambda doesn't return any value (i.e., returns `Unit`), or if the context makes the types clear, we can omit the type annotations:

```kotlin
val printMessage = { message: String ->
    println(message)
}
```

### When `it` is the Only Parameter

In Kotlin, `it` is a keyword used in lambda expressions to refer to the parameter implicitly when there's only one parameter and it hasn't been declared explicitly.

Here's a basic example to illustrate the use of `it`:

```kotlin
val numbers = listOf(1, 2, 3)
numbers.forEach { println(it) }
```

In this example, `forEach` is a higher-order function that takes a lambda expression as its parameter. The lambda `{ println(it) }` is executed for each element in the `numbers` list. Since there's only one parameter to the lambda and its type can be inferred, we use `it` to refer to each element as it's iterated over.

Kotlin lambdas allow for more complex operations too, not just iteration. We can use them for filtering, mapping, and aggregating data in collections. Here's a slightly more complex example using `filter` and `map`:

```kotlin
val doubledPositiveNumbers = numbers.filter { it > 0 }.map { it * 2 }
```

In this case, `filter` removes any numbers that don't satisfy the condition `{ it > 0 }`, and `map` then takes each remaining number, referred to as `it`, and doubles it. The result is a new list assigned to `doubledPositiveNumbers`.

### Optional Parentheses for the Lambda Argument

When we're calling a function with a lambda expression as the last parameter, we have the option to place the lambda outside the parentheses. This convention is particularly common with higher-order functions, like `filter` and `map`, which take functions as parameters.

So, when we see:

```kotlin
numbers.filter { it > 0 }
```

It is equivalent to:

```kotlin
numbers.filter({ it > 0 })
```

This ability to move the lambda outside of the parentheses is particularly handy when the lambda expression is large, making the code easier to read and understand. Here's a more complex example to illustrate:

```kotlin
val result = list.filter { it > 0 }
                 .map { 
                     println("Mapping $it")
                     it * 2 
                 }
```
### Multiple Parameters

Lambdas in Kotlin can also have multiple parameters. In such cases, we can't use `it`, and we need to declare each parameter explicitly, enclosed in parentheses. For example:

```kotlin
val sum = listOf(1 to 2, 3 to 4).map { (a, b) -> a + b }
```

Here, `map` is applied to a list of pairs, and each pair is destructured into `a` and `b` which are then summed.
