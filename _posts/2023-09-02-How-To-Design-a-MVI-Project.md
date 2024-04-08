---
title: How To Design an MVI Project For Android Part 1 - The Model
layout: post
date: 23-09-02
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
  - dependency_injection
  - state_pattern
---

For this project, we're going to use a JSON file as a simple data source and write a program that rolls for the random results on the table defined in the JSON.
# Setting Up a New Project

- File -> New -> New Project -> Empty Activity

# Set Up the Project Structure

- **`src/main/java` or `src/main/kotlin`**: Root directory for our Kotlin source files.
    - **`com/ourapplicationname`**: Our application's package.
        - **`data`**: Directory for data handling and network operations.
            - **`model`**: Contains model classes that represent the structure of our data. Our Kotlin class for the `Model` fits here.
            - **`repository`**: Contains repository classes that abstract the logic of data access and manipulation. They might interact with our model classes and data sources like databases, web services, or our JSON files.
        - **`ui`**: Directory for user interface related classes.
            - **`view`**: Contains activities, fragments, and any other views.
            - **`viewmodel`**: Contains ViewModel classes which hold the logic to manage the UI's data in response to user actions.
        - **`util`**: Utility classes that provide common functionalities across the application.
- **`src/main/assets`**: Where we will keep data sources and other assets.

# The JSON File

In the `assets` directory, I add `lootTable.json` that looks like this:

```json
{
  "tableName": "Dungeon Loot Table",
  "description": "This table determines what loot you find in the dungeon.",
  "results": [
    "Gold coins",
    "Magic sword",
    "Potion of healing"
  ]
}
```

# The Model

### 1. Set Up the Dependencies

We will use `kotlinx` to parse our JSON eventually, so we will have to edit the gradle files.

Under Gradle Scripts, there are two `build.gradle.kts` scripts.  The one you want is marked with `(Module :app)`.

Add the following to the existing sections:

```kotlin
plugins {  
id("org.jetbrains.kotlin.plugin.serialization") version "+"  
}
```


```kotlin
dependencies {  
implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:+")  
}
```

Be sure to sync the project when you're done with editing this file.

Note that I use "+" when setting the version number. This will most often highlight in the editor with an option to use the latest version. You'll notice when looking at a lot of online documentation (such as blogs like this) that they tend to list specific version numbers. You want the latest version though. So use this trick.

Also note that I use Kotlin syntax for my gradle files. Lots of online references will use groovy syntax.

### 2. Create the Model Class

In the `data/model` directory, I add the Kotlin class, `LootTable.kt` that looks like this:

```kotlin
package net.erickveil.mvi_table_roller.data.model  
import kotlinx.serialization.Serializable 

@Serializable
data class LootTable (  
	val tableName: String,  
	val description: String,  
	val results: List<String>  
)
```

In this model:

- `@Serializable` annotation is from `kotlinx.serialization`, indicating that this data class can be serialized and deserialized to and from JSON.
- `LootTable` is the main data class that represents our table, with properties matching the JSON keys: `tableName`, `description`, and `results`.
- `tableName` and `description` are of type `String`.
- `results` is a list of strings, each representing a possible outcome from the table.

