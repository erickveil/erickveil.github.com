---
title: How To Design an MVI Project For Android Part 5 - Putting it Together
layout: post
date: 23-09-10
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
  - dependency_injection
  - state_pattern
---


# Connect ViewModel with UI
Finally, in our Composable function (`LootTableUIEnhanced`), we observe the `LootTableViewState` from our `LootTableViewModel` and update the UI accordingly. 

Let's lay this out in order of the flow of MVI:

### 1.  Give the Composable Function for the View Access to the ViewModel

First, our Button and our Text are the user's Input and Output for this Intent. The ViewModel will be their connection to the business logic.

In `LootTableUIEnhanced` we're going to get rid of the placeholder arguments and give it a `viewModel` object. 
- We're going to use that to get the current state of the UI as represented by the `ViewModel`. 
- The `StateFlow` in the `ViewModel` holds the current state, which includes all the data our UI might need to render itself. 
- When you collect this state within our Composable function, you're telling Compose to re-compose (re-draw) our UI any time the state changes. 
- This state could be the result of processing various intents, but it's not the intent itself.

Change the function arguments:
```kotlin
fun LootTableUIEnhanced(viewModel: LootTableViewModel) {  
```

### 2. Capture The Intent From the User

Change the Button Code:
```kotlin
Button( 
	onClick = { 
		viewModel.processIntent(LootTableIntent.RollLootTable) 
	}, 
```

The `Button` Composable within `LootTableUIEnhanced` captures the user's intent to "roll the loot table" through its `onClick` listener. 
When the user presses the button, this listener invokes the `processIntent` method on the `ViewModel` with an argument that signifies the specific action or intent (`LootTableIntent.RollLootTable` in this case).

In the `processIntent` function, we basically have our "switch", the `when(intent) { is... }` statement where we look at the intent, then call the appropriate business logic.

```kotlin
fun processIntent(intent: LootTableIntent) {
    when (intent) {
        is LootTableIntent.RollLootTable -> rollLootTable()
    }
}
```

So now we have our Intent, `RollLootTable` and we call `rollLootTable()`.

```kotlin
private fun rollLootTable() {  
	_state.value = _state.value.copy(  
	isLoading = true,  
	resultText = "Magic sword"  
	)   
}
```

Here, we do some work (in this case, just some example values are set) and the result is given to the state. 
- **`_state.value`**: This accesses the current value of the `_state` `MutableStateFlow`.
- **`.copy(isLoading = true, resultText = "Magic sword")`**: Since `LootTableViewState` is a data class, you can use the `copy` method to create a new instance of it with some properties changed. This line creates a new `LootTableViewState` where `isLoading` is set to `true`, and `resultText` is set to "Magic sword", while keeping all other properties the same as in the current state.
- **Assignment**: The new state is then assigned back to `_state.value`, effectively updating the state.

We had defined the state in this `ViewModel` class using our class, `LootTableViewState`:


```kotlin
private val _state = MutableStateFlow(LootTableViewState())  
val state: StateFlow<LootTableViewState> = _state.asStateFlow()
```

- Here, `_state` is a private mutable state flow that you'll use within our `ViewModel` to update the state. 
- Meanwhile, `state` is the public, read-only version of this state flow that the UI components can collect and observe for changes.

### 3. Update the View With the State Changes

Now, back at our View's Composable function, `LootTableUIEnhanced` where we want the value to update, we use the public access to the state to change the text that the user sees.

Change the Text source:
```kotlin
Text(  
	text = state.value.resultText,
```
This is the point where we observe the changes in the state stored in the ViewModel. It's "declarative" in that the UI automatically updates in response to this change (without "imperative" commands like `setText()`).

We had defined the `state` before we started defining **Composables** (the visual UI elements usually referred to as "Controls" in other languages) which has access to the same ViewModel as the one we used in the Button. 

Add at the top of the function:
```kotlin
val state = viewModel.state.collectAsState()
```

This brings us full circle from button click to setting Intent, to doing the work with the Model required by that Intent, and then updating the View with the changes.

### 4. Update the MainActivity So That It Uses the View Correctly

Finally, we need to make sure that `MainActivity` is calling `LootTableUIEnhanced` with the correct argument, now that we changed the signature of that function.

Change the object passed to the function:
```kotlin
Surface() {  
	LootTableUIEnhanced(viewModel())  
}
```

The `viewModel()` method is part of Compose, and in this case it creates a new ViewModel to use.
It automatically infers the type of `ViewModel` thanks to that function requiring a specific subclass - in this case `LootTableViewModel`. It also scopes that object to the activity it is created in (`MainActivity` in this case) so that it is preserved.

In situations where type inference might not work as expected or when you want to be explicit, you can specify the `ViewModel` class directly:

```kotlin
val myViewModel: LootTableViewModel = viewModel()
```

Or, when passing it directly as a parameter without assigning it to a variable:

```kotlin
LootTableUIEnhanced(viewModel = viewModel())
```
