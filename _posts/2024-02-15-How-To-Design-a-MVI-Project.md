---
title: How to Design MVI in Android
date: 24-02-15
layout: post
tags:
  - android
  - architecture
  - kotlin
  - jetpack
---

I know I posted an in depth series of blog posts about this, but it's so long that it's a bit hard to follow. I think I've been able to simplify the concept better here. So I've removed the 7 posts (!) I wrote on the subject, and replaced them with this one, consolidated post.

Here is a simplified breakdown of the MVI architecture. 
# Model

The state class. Make it a data class:

```kotlin
data class BrownNoiseState(
    val isPlaying: Boolean = false
)
```

# Intent

A sealed class:

```kotlin
sealed class BrownNoiseIntent {
    object TogglePlayback : BrownNoiseIntent()
}
```

# ViewModel

A standard class that extends `ViewModel`:

```kotlin
import kotlinx.coroutines.flow.MutableStateFlow  
import kotlinx.coroutines.flow.StateFlow
import androidx.lifecycle.ViewModel  
import androidx.lifecycle.viewModelScope  
import kotlinx.coroutines.launch  
  
class BrownNoiseViewModel : ViewModel() {

}
```

The ViewModel will always track the mutable state:

```kotlin
private val _state = MutableStateFlow(BrownNoiseState()) 
val state: StateFlow<BrownNoiseState> get() = _state
```

Then you will have the public method to process your intents:

```kotlin
fun processIntent(intent: BrownNoiseIntent) { 
	when (intent) { 
		is BrownNoiseIntent.TogglePlayback -> togglePlayback() 
	} 
}
```

Then the private methods where you do the work.
- These work methods are triggered by the public intent processing method.
- Their final effects are to change values in the state.

## `ViewModelScope::launch`

The `launch` function is used to start a new coroutine.

Coroutines launched in `viewModelScope` are automatically canceled when the ViewModel is cleared, such as when the associated UI component (like an Activity or Fragment) is destroyed. This helps in preventing memory leaks and ensuring that ongoing operations do not continue when they are no longer needed.

By using `viewModelScope.launch`, we can perform concurrent operations without blocking the main thread. This keeps us from freezing the UI while we do work.

Finally, `viewModelScope` is lifecycle-aware, so it automatically cancels the coroutine if the ViewModel is cleared. This prevents potential issues like memory leaks and ensures that no unnecessary operations are performed when the UI component is no longer in use.

Example:

```kotlin
private fun togglePlayback() { 
	viewModelScope.launch { 
		val newState = !_state.value.isPlaying 
		_state.value = BrownNoiseState(isPlaying = newState) 
		if (newState) { 
			startBrownNoise() 
		} else { 
			stopBrownNoise() 
		} 
	} 
}
```

# View

Set up `MainActivity` like so:

```kotlin
import androidx.lifecycle.compose.collectAsStateWithLifecycle  
import net.erickveil.calmsound.intent.BrownNoiseIntent  
import net.erickveil.calmsound.model.BrownNoiseState  
import net.erickveil.calmsound.ui.theme.CalmSoundTheme 
import androidx.activity.viewModels
  
class MainActivity : ComponentActivity() {  
  
    private val viewModel: BrownNoiseViewModel by viewModels()  
  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContent {  
            CalmSoundTheme {  
                val state by viewModel.state.collectAsStateWithLifecycle()  
                BrownNoisePlayerScreen(  
                    state = state,  
                    onTogglePlayback = {  
                        viewModel.processIntent(BrownNoiseIntent.TogglePlayback)  
                    }  
                )  
            }  
        }    
    }  
  
    @Composable  
    fun BrownNoisePlayerScreen(
        state: BrownNoiseState,  
        onTogglePlayback: () -> Unit  
    ) {  
        Button(onClick = onTogglePlayback) {  
            Text(if (state.isPlaying) "Stop Brown Noise" else "Play Brown Noise")  
        }  
    }  
}
```

#### `private val viewModel` 
This gets us the ViewModel in a lifecycle aware way.

#### `state by viewModel.state.collectAsStateWithLifecycle()` 
This allows us to capture the current state in a lifecycle aware way. This lets us stop collecting the state when the composable is not active. This also allows any change in the **state** to be reflected in the Composable.

#### `onTogglePlayback: () -> Unit`
This function parameter is a lambda used as a callback.

Here, in the `onCreate` we define what happens when the button is pressed on the backend, where we mess with intent and trigger the work to be done in the **ViewModel**:

```kotlin
onTogglePlayback = {  
	viewModel.processIntent(BrownNoiseIntent.TogglePlayback)  
}  
```

Here we connect this to the button click and define what gets done in the **View**:

```kotlin
Button(onClick = onTogglePlayback) {
	Text(if (state.isPlaying) "Stop Brown Noise" else "Play Brown Noise")  
}  
```

So we have a separation of concerns. The View code is not mixed up with the ViewModel code.

For a full example of an MVI project, take a look at my fully functional [CalmSound](https://github.com/erickveil/CalmSound) app's sourcecode. You can also find this app on the Google Play Store.
