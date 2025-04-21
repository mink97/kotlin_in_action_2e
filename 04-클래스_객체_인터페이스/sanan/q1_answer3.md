```kotlin
@JvmInline
value class Amount(val value: Int) {
    init {
        require(value >= 0) { "Amount must be non-negative" }
    }
}
```