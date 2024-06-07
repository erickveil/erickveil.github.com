---
title: How To Design an MVI Project For Android Part 3 - The Intent
layout: post
date: 23-09-06
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
  - dependency_injection
  - state_pattern
---


# Define the Intent

In the Model-View-Intent (MVI) architecture, after setting up the Model and the View, the next step indeed involves defining Intents. Intents in MVI are responsible for representing the user's intentions, essentially the actions that the user can perform on the UI that might require some data processing or logic execution, such as fetching data, updating the UI, etc.

For our application, where users interact with a "Roll on Loot Table" button to get a result, you'd define an Intent that captures this user action. Here's how you can approach it:

### Defining Intents

1. **Identify User Actions**: Start by identifying all the possible actions users can perform. In our case, it's primarily the action of "rolling" to get a loot result.

2. **Create an Intent Sealed Class**: A sealed class in Kotlin is a good way to define a restricted class hierarchy that represents all possible Intents our View can emit. For our loot table application, this might look something like:

    ```kotlin
    sealed class LootTableIntent {
        object RollLootTable : LootTableIntent()
        // Add more Intents here as our application grows
    }
    ```

So, think of this in terms of setting up the **state** objects for a state pattern:
- We have a list of states (actually a list of intents) called `LootTableIntent`.
- Every **intent** (every action a user might take with a UI, intending things to happen) is one of the state objects.
- Each subclass of `LootTableIntent` is going to be used like each **state** would be.
- So here we just have one intent/state, because this is a simple example.
- That intent is `RollLootTable`
- The notation `object RollLootTable : LootTableIntent()` is basically a shorthand for:
	- Subclass the `LottTableIntent`, creating a new intent/state.
	- Do it in a way that makes this a singleton.
So it's basically here a supper powered `enum` value that better follows the state pattern than an enum would (which is why we have the state pattern).

The `sealed` class makes it so that all of the values for this state machine are going to be right here in this file only.

