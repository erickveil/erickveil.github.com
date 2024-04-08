---
title: How To Design an MVI Project For Android Part 4 - The ViewState and the ViewModel
layout: post
date: 23-09-08
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
  - dependency_injection
  - state_pattern
---

### Handling Intents in Our Architecture

Once you have our Intents defined, the next steps in the MVI architecture involve creating a ViewModel that listens to these Intents, performs the necessary data processing (fetching the loot results in this case), and then publishes a new state to the UI.

### Implementing the ViewModel

Our ViewModel will act as the intermediary that handles Intents, interacts with the Model to retrieve or manipulate data, and then generates a new ViewState that reflects the result of these operations. Here's a simplified version of what this might look like:

#### 1. Define a ViewState 
First, define a data class that represents the state of our UI at any given time. This class will include all the necessary information to render our UI.

```kotlin
data class LootTableViewState(
	val isLoading: Boolean = false,
	val resultText: String = "Result appears here",
	val error: String? = null
)
```

#### 2. Create a ViewModel
Implement a ViewModel that handles the Intents and updates the ViewState accordingly.

```kotlin
class LootTableViewModel : ViewModel() {
	private val _state = MutableStateFlow(LootTableViewState())
	val state: StateFlow<LootTableViewState> = _state.asStateFlow()

	fun processIntent(intent: LootTableIntent) {
		when (intent) {
			is LootTableIntent.RollLootTable -> rollLootTable()
		}
	}

	private fun rollLootTable() {
		// Simulate fetching data and updating the state
		_state.value = _state.value.copy(
			isLoading = true,
			// Simulate fetching a random result
			resultText = "Magic sword"
		)
		// Here, you would actually fetch the data from our model and update
		// the state accordingly
	}
}
```

Breakdown:

Here we set up the state machine:
```kotlin
 private val _state = MutableStateFlow(LootTableViewState())
 val state: StateFlow<LootTableViewState> = _state.asStateFlow()
```

Here we look at the current intent/state and call a method depending on what it is.
If we had different intents, they would each be handled by a separate `is` statement.

```kotlin
when (intent) {
	is LootTableIntent.RollLootTable -> rollLootTable()
}
```

Now here, we would be interacting with the Model and rolling on the table. We don't actaully do that yet. This is just a palceholder:

```kotlin
private fun rollLootTable() { ...
```
