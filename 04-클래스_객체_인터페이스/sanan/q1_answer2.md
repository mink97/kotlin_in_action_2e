```kotlin
data class Calculator(val value: Int = 0) {
    fun increment(amount: Int): Calculator {
        return this.copy(
            value = value + amount
        )
    }
}
```