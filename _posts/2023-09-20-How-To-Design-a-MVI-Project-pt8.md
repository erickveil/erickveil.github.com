---
title: How To Design an MVI Project For Android Part 8 - Conclusion
layout: post
date: 23-09-20
tags:
  - kotlin
  - mvi
  - android
  - jetpack
  - architecture
  - dependency_injection
  - state_pattern
---


# MVI and the Files We Created

Now we have a working table roller, where the table can be described using a JSON file.

Our Android project structure, organized according to the Model-View-Intent (MVI) architecture, aligns each component with one of the MVI layers. Here's how each file fits into the MVI pattern:

### Model
The Model is responsible for the data-related logic, such as fetching, storing, and manipulating data.

- **`/data/model/LootTable.kt`**: Represents the data structure of our loot table. It's the data model part of the Model layer.
- **`/data/repository/LootRepository.kt`**: Acts as the data source layer that would fetch, cache, and manage our loot data (e.g., reading the `lootTable.json` file). It's part of the Model layer, serving as an intermediary between the raw data and the ViewModel.

### View
The View is responsible for presenting data to the user and capturing user inputs.

- **`/ui/view/LootTableView.kt`**: Defines the UI components (Composables) that display the loot table and button to the user. It's part of the View layer in MVI.
- **`/MainActivity.kt`**: While not a Composable itself, it sets up the content view for our app and could be considered part of the View layer since it's where our Composables are hosted.
- **`/TableRollerApp.kt`**: Likely the application class. While not directly part of the MVI pattern, it's essential for initializing app-wide components, like Hilt for dependency injection.

### Intent
The Intent layer captures the intentions to change the state of the app based on user input or other actions.

- **`/ui/intent/LootTableIntent.kt`**: Defines the actions (intents) that can be taken in the app, like rolling the loot table. It directly corresponds to the Intent layer in MVI.

### ViewModel
Though not traditionally separate in the MVI acronym, the ViewModel acts as a bridge between the View and Model, handling state and intents.

- **`/ui/viewmodel/LootTableViewModel.kt`**: Contains the business logic responding to user intents, fetching data from the repository, and preparing it to be displayed by the View. It processes intents, updates the state, and thus serves as a crucial part of both the Intent and Model layers.

### ViewState
ViewState represents the current state of the UI, which is a concept closely tied to both the View and the ViewModel.

- **`/ui/viewstate/LootTableViewState.kt`**: Defines the various states of the UI (e.g., loading, success, error) based on the data from the ViewModel. It's closely related to both the View (as it dictates what the View displays) and the ViewModel (as it's shaped by the data processing logic within the ViewModel).

# Conclusion
This project structure reflects a clear separation of concerns as prescribed by the MVI architecture, with each component playing a specific role:

- **Model**: `LootTable.kt`, `LootRepository.kt`
- **View**: `LootTableView.kt`, `MainActivity.kt`
- **Intent**: `LootTableIntent.kt`
- **Bridge (ViewModel and ViewState)**: `LootTableViewModel.kt`, `LootTableViewState.kt`

This organization facilitates understanding, maintaining, and scaling our application by clearly delineating responsibilities within our codebase.
