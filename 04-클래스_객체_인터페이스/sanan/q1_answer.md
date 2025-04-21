```kotlin
sealed interface UiState {
    object Loading: UiState
    data class Loaded(val title: String, val description: String)
    data class Error(val message: String?)
}
```