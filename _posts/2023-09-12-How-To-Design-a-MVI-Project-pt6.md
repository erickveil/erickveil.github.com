---
title: How To Design an MVI Project For Android Part 6 - The Repository
layout: post
date: 23-09-12
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
  - dependency_injection
  - state_pattern
---

# Working With the Actual Data Model

At this point, we display a constant string when the user pushes the button. But we want to:

- Load the JSON into the Model.
- Chose a random item from the List of options.
- Use that as the string displayed in our output box.

### Create the Repository

The repository is the class that we will use to manage our Data Model. From here, we will handle loading the JSON into a Model class.

```kotlin
class LootRepository(private val context: Context) {  

	fun getLootTable(): LootTable? {  
		val jsonString = loadJsonFromAssets("lootTable.json")  
		return jsonString?.let { parseJsonToLootTable(it) }  
	}  
	  
	private fun loadJsonFromAssets(fileName: String): String? {  
		return try {  
			context.assets.open(fileName).bufferedReader().use { it.readText() }  
		} catch (ioException: IOException) {  
			ioException.printStackTrace()  
			null  
		}  
	}  
	  
	private fun parseJsonToLootTable(jsonString: String): LootTable? {  
		return try {  
			Json.decodeFromString(serializer(), jsonString)  
		} catch (exception: SerializationException) {  
			exception.printStackTrace()  
			null  
		}  
	}  
}
```

Since we set up the `LootTable` data model as serializable, we can just continue using kotlinx to parse.

We will need the `Context` object so that we can load the JSON resource from the `assets` directory.

### Use the Repository in the ViewModel

Next we modify the ViewModel to load the JSON using the Repository. Since we only need to load it once and reference it, we can do that in the class's `init` method:

First, we will need to modify our ViewModel to accept the `Context`.
```kotlin
class LootTableViewModel(context: Context): ViewModel() {
```

Then add some class members to instantiate the Repository and hold our Data Model where we will be referencing it.
```kotlin
private val repository = LootRepository(context)  
private var lootTable: LootTable? = null
```

Next we create a method to load it.

We will use `viewModelScope.launch { }` to put the file loading into a coroutine. This will load the file asynchronously. It's possible that you'd want to create a state where `isLoading` is manipulated so the rest of the program knows it's safe to access the data. If the data source is particularly large, you might want to do this on a background thread. 

We're going to keep this simple here for this case though:
```kotlin
private fun loadLootTable() {  
	viewModelScope.launch {  
		lootTable = repository.getLootTable()  
	}  
}
```

Finally, we just run this in the `init`.
```kotlin
init {
	loadLootTable()
}
```

### Roll on the Table

Let's put the actual roll in its own method.

If we were certain of the presence of data (we are not, since the JSON can have *anything*, but let's suppose) then we can just select a random item from the list with
```kotlin
lootTable.results.random()
```

But we want to make sure we're not performing operations on null objects, so we will use a lambda with `takeIf { it }` which will return `null` if any of the optional values are missing, or execute the `random()` method if all is well.
```kotlin
private fun getRandomLootItem(): String? {  
	return lootTable?.results?.takeIf { it.isNotEmpty() }?.random()  
}
```

Now we can remove the placeholder code and select a random item from the table instead:
```kotlin
private fun rollLootTable() {  
	val randomItem = getRandomLootItem()  
	_state.value = _state.value.copy(  
		isLoading = false,  
		resultText = randomItem ?: "No loot found"  
	)  
}
```

This function updates the `_state` with a new value, setting `isLoading` to `false` (assuming you're using it to show a loading indicator) and `resultText` to the randomly selected item. If no item is found (for example, if `results` is empty or `lootTable` is not loaded), it sets `resultText` to "No loot found".
#### Note on Safety and Randomness

- **Null Safety**: The use of the safe call operator (`?.`) and the Elvis operator (`?:`) ensures that the function gracefully handles cases where `lootTable` might not be initialized or `results` is empty.
    
- **Randomness**: The `random()` function provides an easy and concise way to get a random element. Ensure our list has elements before calling `.random()` to avoid exceptions. The check `it.isNotEmpty()` ensures this safety.
