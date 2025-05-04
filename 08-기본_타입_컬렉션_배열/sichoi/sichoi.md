# 원시 타입과 기본 타입

## 정수 , 부동소수점 수 , 문자 , 불리언 값을 원시 타입으로 표현

**자바(Java)**

- **원시 타입**과 **참조 타입**을 **명확히 구분**
    - 예: `int` (원시 타입) vs `Integer` (래퍼 클래스)
- **제네릭, 컬렉션**에는 원시 타입 사용 불가 → 반드시 래퍼 타입 사용
    
    ```java
    List<Integer> list = Arrays.asList(1, 2, 3); // OK
    List<int> list = ...; // ❌ 컴파일 에러
    ```
    
- **메서드 호출 불가**: 원시 타입은 객체가 아니므로 메서드 호출 불가

**코틀린(Kotlin)**

- **원시 타입과 래퍼 타입을 구분하지 않음** → **모두 하나의 타입**으로 통합
    - 예: `Int`, `Double`, `Boolean`, `Char` 등은 모두 **객체처럼 보이지만**,
        
        **실행 시점에서는 효율적으로 컴파일됨**
        
- 예:
    
    ```kotlin
    val n: Int = 1
    val list: List<Int> = listOf(1, 2, 3)
    ```
    

### 실행 시 최적화

- 컴파일러가 가능한 한 자바의 **원시 타입(`int`, `float` 등)** 으로 자동 변환
- 다만 **제네릭(예: `List<Int>`) 사용 시에는 래퍼 객체 (`Integer`) 사용**

### 코틀린의 타입 목록

| 타입 종류 | Kotlin 타입 |
| --- | --- |
| 정수형 | `Byte`, `Short`, `Int`, `Long` |
| 실수형 | `Float`, `Double` |
| 문자형 | `Char` |
| 불리언형 | `Boolean` |

## 양수를 표현하기 위해 모든 비트 범위 사용： 부호 없는 숫자 타입

### 부호 없는 타입이 필요한 이유

- **모든 비트를 양수 표현에 사용**하고 싶을 때 유용
    - 예: **비트 조작**, **픽셀 처리**, **바이너리 파일 파싱**, **네트워크 프로토콜**
- 자바에서는 지원하지 않지만, **코틀린은 이를 인라인 클래스로 추상화**하여 지원

### 코틀린의 부호 없는 타입 목록

| 타입 | 크기 | 값 범위 |
| --- | --- | --- |
| `UByte` | 8비트 | `0` ~ `255` |
| `UShort` | 16비트 | `0` ~ `65,535` |
| `UInt` | 32비트 | `0` ~ `4,294,967,295` |
| `ULong` | 64비트 | `0` ~ `18,446,744,073,709,551,615` |

> 부호 있는 타입과 동일한 메모리를 사용하지만 범위를 오른쪽으로 시프트
> 

### 동작 방식 (성능 관련)

- 실제 구현은 `Int`, `Long` 같은 **부호 있는 타입을 내부에 저장**
- `@JvmInline`을 사용한 **인라인 클래스** 기반 → 오버헤드 없음
- 즉, `UInt`는 내부적으로 `Int`지만 **API 수준에서 다르게 동작**

### 주의할 점

- JVM은 부호 없는 타입을 **직접적으로 지원하지 않음**
- 코틀린의 부호 없는 타입은 **정수의 모든 비트가 의미 있는 경우에만 사용**
- 일반적인 상황에서는 **`Int`를 사용하고 음수 여부를 검사하는 것이 더 직관적**

### 예시

```kotlin
fun showProgress(progress: UInt) {
    val percent = progress.coerceAtMost(100u)
    println("We're $percent% done!")
}

showProgress(146u)
// 출력: We're 100% done!
```

## 널이 될 수 있는 기본 타입： Int?. Boolean? 등

### 기본 규칙

- **null은 자바의 원시 타입(`int`, `double`, 등)에 저장할 수 없음**
- 따라서 코틀린에서 `Int?`, `Double?`, `Boolean?` 등 **널이 될 수 있는 원시 타입**을 사용하면
    
    → 자바의 **래퍼 타입(Integer, Double 등)** 으로 컴파일됨
    

### 예제 요약

```kotlin
data class Person(val name: String, val age: Int? = null) {
    fun isOlderThan(other: Person): Boolean? {
        if (age == null || other.age == null) return null
        return age > other.age
    }
}
```

- `Int?`는 자바에서는 `Integer`
- null 비교 후에만 `>` 연산 가능(**스마트 캐스트** 발생)

### 제네릭과 박싱(Boxing)

- 자바/JVM은 **제네릭에 원시 타입을 직접 사용할 수 없음**
- 따라서 `listOf(1, 2, 3)` → 내부적으로 `List<Integer>` 사용됨

```kotlin
val listOfInts: List<Int> = listOf(1, 2, 3) // List<Int>지만 실제론 List<Integer>
```

- null이 없어도 **박스 타입 사용은 불가피** (JVM 한계 때문)
- **변수/프로퍼티에 null 가능성만 판단하면 됨**
    
    → 내부 표현(Int vs Integer)은 컴파일러가 자동 처리
    
- 성능이 중요하면: `Array<Int>` 대신 `IntArray` 같은 **원시 배열** 또는 **전용 라이브러리** 고려

## 수 변환

### **자동 변환 금지**

- 코틀린은 **숫자 타입 간 자동 변환을 허용하지 않음**
- 예: `Int → Long` 도 명시적으로 `.toLong()` 호출 필요

```kotlin
val i = 1
val l: Long = i // ❌ 컴파일 오류
val l2: Long = i.toLong() // ✅
```

### **명시적 변환 함수**

| 함수 예시 | 의미 |
| --- | --- |
| `toByte()` | 강제로 Byte로 변환 |
| `toShort()` | Short로 변환 |
| `toInt()` | Int로 변환 |
| `toLong()` | Long으로 변환 |
| `toFloat()` | Float로 변환 |
| `toDouble()` | Double로 변환 |
| `toChar()` | 문자로 변환 (주의: Int가 문자 코드로 간주됨) |

### **박스 타입 비교 시 주의 사항**

- 서로 다른 박스 타입은 equals 비교 시 false
    
    → `Integer(42) != Long(42)`
    
- 따라서 `x in listOf(1L, 2L)` 같은 건 **명시적 변환 필수**

### **숫자 리터럴 종류**

| 리터럴 형태 | 예시 | 설명 |
| --- | --- | --- |
| `L` 접미사 | `123L` | Long |
| `f`, `F` 접미사 | `3.14f` | Float |
| `e`, `E` 표기 | `1.2e3` | 지수형 Double |
| `0x`, `0X` | `0xFF` | 16진수 |
| `0b`, `0B` | `0b1010` | 2진수 |
| `u`, `U`, `UL` | `123U`, `123UL` | 부호 없는 정수 |

### **산술 연산은 예외적으로 자동 변환**

- 리터럴을 쓸 때는 컴파일러가 타입을 추론해서 문제 없음

```kotlin
val b: Byte = 1
val l = b + 1L // 결과는 Long
```

- 함수 인자도 리터럴이면 자동 처리

```kotlin
fun printALong(l: Long) = println(l)
printALong(42) // OK, 42는 자동 Long 추론
```

### **오버플로우/언더플로우 발생 가능**

- 오버/언더플로우는 Java와 마찬가지로 **검사하지 않음**

```kotlin
println(Int.MAX_VALUE + 1) // -2147483648 (오버플로우)
println(Int.MIN_VALUE - 1) // 2147483647 (언더플로우)
```

### **문자열 → 숫자 변환**

- 예외 발생 가능:

```kotlin
"42".toInt()           // 42
"seven".toInt()        // ❌ NumberFormatException
```

- 안전한 변환 (nullable 반환):

```kotlin
"seven".toIntOrNull()  // null
```

### **문자열 → 불리언 변환**

| 함수 | 설명 |
| --- | --- |
| `toBoolean()` | "true" (대소문자 무시)만 true, 나머지 false |
| `toBooleanStrict()` | 정확히 "true"/"false"만 허용, 아니면 예외 |

## Any와 Any?: 코틀린 타입 계층의 뿌리

### `Any`: 모든 **널이 될 수 없는 타입**의 최상위 타입

- 자바의 `Object`와 유사하지만, **원시 타입(코틀린의 기본 타입)까지 포함**.
- 기본적으로 `null`을 포함할 수 없음 → `Any?`로 선언해야 `null` 대입 가능.
- 자바의 `Object`를 넘겨받는 경우는 `Any!` (플랫폼 타입)로 해석됨.
- `toString()`, `hashCode()`, `equals()`는 `Any`에 정의돼 있음.
- `wait()`, `notify()`는 `java.lang.Object`에서만 사용 가능 → 명시적 캐스팅 필요.

```kotlin
val a: Any = 42 // 자동 박싱됨
val b: Any? = null // null도 대입 가능
```

### `Unit`: 자바의 `void` 대응

- 반환 값이 **의미 없을 때** 사용하는 타입.
- 실제로는 `Unit`이라는 **단 하나의 인스턴스**를 가지는 일반적인 타입.
- 타입 인자로도 사용 가능.

```kotlin
fun printSomething(): Unit {
    println("Hello") // return Unit 생략 가능
}

val result: Unit = printSomething() // 값도 가질 수 있음
```

- 자바와 달리 **값으로 사용 가능**하며, **제네릭 타입**으로도 활용 가능.

### `Nothing`: **절대 값을 반환하지 않는 함수**의 반환 타입

- 예외를 던지거나, 무한 루프에 빠지거나, 절대 끝나지 않는 함수에 사용.
- `Nothing`을 반환하는 함수는 **정상 종료되지 않음**을 컴파일러가 알게 해줌.

```kotlin
fun fail(message: String): Nothing {
    throw IllegalStateException(message)
}

val address = company.address ?: fail("No address")
// 이후 address는 null이 아님을 보장받음

```

- 엘비스 연산자와 같이 쓰면 **컴파일러 null 추론 최적화**에 도움.

| 타입 | 설명 | null 포함 | 사용 예 |
| --- | --- | --- | --- |
| `Any` | 모든 널이 아님 타입의 조상 | ❌ (`Any?` 필요) | 값 보관, 다형성 인자 |
| `Unit` | 반환 값 없음 (`void` 유사) | ❌ | 함수 반환, 제네릭 |
| `Nothing` | 절대 반환되지 않는 함수 타입 | N/A | 예외, 무한 루프 |

# 컬렉션과 배열

## 널 가능성과 컬렉션 타입

| 선언 | 의미 |
| --- | --- |
| `List<Int?>` | 리스트 안에 null이 들어갈 수 있음 (`1, null, 3`) |
| `List<Int>?` | 리스트 자체가 null일 수 있음 (`null`, `listOf(...)`) |
| `List<Int?>?` | 리스트도 null일 수 있고, 원소도 null일 수 있음 |

### 리스트의 요소가 null 가능

```kotlin
fun readNumbers(text: String): List<Int?> {
    return text.lineSequence()
        .map { it.toIntOrNull() }  // 각 줄을 Int로 변환 (실패 시 null)
        .toList()
}
```

- 반환 타입: `List<Int?>`
    
    → 리스트는 존재하되, **원소 일부가 null일 수 있음**
    

### null 값이 포함된 리스트 다루기

```kotlin
fun addValidNumbers(numbers: List<Int?>) {
    val validNumbers = numbers.filterNotNull() // null 제거
    println("합계: ${validNumbers.sum()}")
    println("무시된 수: ${numbers.size - validNumbers.size}")
}
```

- `filterNotNull()`은 null을 제거한 새로운 `List<Int>` 반환
- 이 덕분에 이후 null 체크 없이 안전하게 연산 가능

## 읽기 전용과 변경 가능한 컬렉션

| 컬렉션 인터페이스 | 특징 |
| --- | --- |
| `Collection<T>` | **읽기 전용(Read-only)** 컬렉션. 원소 조회만 가능 |
| `MutableCollection<T>` | **변경 가능(Mutable)** 컬렉션. 원소 추가/삭제 가능 |
- `Collection`에는 `add()`, `remove()` 없음 → 변경 불가
- `MutableCollection`은 `Collection`을 확장하며 변경 기능 제공

### 기본 예시

```kotlin
fun <T> copyElements(source: Collection<T>, target: MutableCollection<T>) {
    for (item in source) {
        target.add(item)
    }
}

val source: Collection<Int> = arrayListOf(3, 5, 7)
val target: MutableCollection<Int> = arrayListOf(1)
copyElements(source, target)
// 결과: [1, 3, 5, 7]
```

- `source`는 읽기 전용 → 원소 조회만 가능
- `target`은 변경 가능 → 원소 추가 가능

### ⚠️ 주의: 읽기 전용 ≠ 불변

```kotlin
val list: List<String> = mutableListOf("a", "b", "c")
```

- `list`는 `List` 타입이지만 실제 객체는 `MutableList`
- 따라서 다른 참조가 `mutableList`로 접근할 경우 **변경 가능**

### 동시성 주의사항

- 읽기 전용 컬렉션이 **스레드 안전하지 않음**
- 다른 스레드에서 `mutable` 참조를 통해 내용을 바꿀 수 있음
- 동시 접근 상황에서는 적절한 **동기화** 필요

### 진짜 불변 컬렉션을 원한다면?

- 표준 라이브러리에는 없음
- → 대신 `kotlinx.collections.immutable` 라이브러리 사용
    
    [🔗 GitHub 링크](https://github.com/Kotlin/kotlinx.collections.immutable)
    

## 코틀린 컬렉션과 자바 컬렉션의 연관성

### 기본 원칙

- **코틀린의 컬렉션은 자바 컬렉션과 호환**
- `List`, `Set`, `Map` 등 코틀린 컬렉션은 내부적으로 **자바 컬렉션 인터페이스를 구현**
- 따라서 자바 <-> 코틀린 컬렉션 간 **변환 없이 직접 사용 가능**.

### 읽기 전용 vs 변경 가능 구조

| 구분 | 인터페이스 | 설명 |
| --- | --- | --- |
| 읽기 전용 | `Collection<T>`, `List<T>`, ... | 원소 접근만 가능, `add`, `remove` 없음 |
| 변경 가능 (Mutable) | `MutableCollection<T>`, ... | 읽기 기능 + `add`, `remove`, `clear` 가능 |
- 코틀린은 **읽기/쓰기 인터페이스를 명확히 분리**.
- 하지만 내부 구현체는 모두 자바의 `ArrayList`, `HashSet` 등 변경 가능한 클래스

### ⚠️ 주의: 읽기 전용 ≠ 불변

- `listOf()`로 만든 `List<String>`도 내부적으로는 `ArrayList`일 수 있음.
- 읽기 전용 타입이라고 해서 **내용이 변경 불가능하다고 보장되지 않음**.
- **미래 버전**에서 완전한 불변 컬렉션을 반환할 수도 있으므로, 구현에 의존하지 말 것.

| 컬렉션 타입 | 읽기 전용 함수 | 변경 가능 함수 |
| --- | --- | --- |
| List | `listOf()` | `mutableListOf()`, `arrayListOf()` |
| Set | `setOf()` | `mutableSetOf()`, `hashSetOf()` |
| Map | `mapOf()` | `mutableMapOf()`, `hashMapOf()` |

### ⚠️ 자바와 혼용할 때 주의점

- **코틀린 `List<T>`를 자바로 넘기면 자바는 이를 자유롭게 수정할 수 있다.**
    - 코틀린 컴파일러는 이를 막지 못함.
- 예시:
    
    ```kotlin
    fun printlnUppercase(list: List<String>) {
        println(CollectionUtils.uppercaseAll(list)) // 자바에서 list 내용 변경
        println(list.first()) // 변경된 내용 노출
    }
    ```
    
- **자바에서는 null 삽입도 가능** → `List<String>`에 `null`이 들어갈 수 있음!

## 핵심 요약

- 코틀린 컬렉션은 자바 컬렉션과 100% 호환되며, 변환 없이 인자로 전달 가능.
- **읽기 전용 타입은 변경이 불가능하다는 뜻이 아님**.
- 자바에 컬렉션을 넘기는 경우, **자바 쪽에서의 변경 가능성/nullable 삽입 가능성**을 반드시 고려해야 함.

## 자바 컬렉션과 코틀린 플랫폼 타입

### **플랫폼 타입**

- 자바에서 온 타입은 **널 가능성 정보가 없으므로** 코틀린에서는 플랫폼 타입(`Type!`)으로 처리된다.
- **널이 될 수 있음/없음 둘 다 허용**되며, 사용자가 적절히 판단해야 한다.

### 자바 컬렉션 → 코틀린에서는 플랫폼 타입

- 자바의 `List<T>` → 코틀린에서는 `List<T>!`
- 코틀린은 이 타입을 **`List<T>` (읽기 전용)** 또는 **`MutableList<T>` (변경 가능)** 로 해석할 수 있다.
- 컬렉션을 오버라이드하거나 타입을 명시해야 할 때, 다음 사항을 명확히 판단해야 한다:

### 판단 기준

1. 컬렉션 자체가 `null`이 될 수 있는가?
2. 컬렉션 안의 **원소**가 `null`이 될 수 있는가?
3. 컬렉션의 **내용을 수정할 필요가 있는가?**

### 예시 1: `FileContentProcessor` 자바 인터페이스

```java
interface FileContentProcessor {
    void processContents(File path, byte[] binaryContents, List<String> textContents);
}
```

**코틀린 구현**

```kotlin
class FileIndexer : FileContentProcessor {
    override fun processContents(
        path: File,
        binaryContents: ByteArray?,
        textContents: List<String>?
    ) {
        // null 가능성, 읽기 전용 리스트
    }
}
```

- `textContents`는 null 가능 (`List<String>?`)
- `List<String>`은 **읽기 전용**, 원소는 non-null

### 📌 예시 2: `DataParser<T>` 자바 인터페이스

```java
interface DataParser<T> {
    void parseData(String input, List<T> output, List<String> errors);
}
```

**코틀린 구현**

```kotlin
class PersonParser : DataParser<Person> {
    override fun parseData(
        input: String,
        output: MutableList<Person>,          // 변경 가능
        errors: MutableList<String?>          // 원소는 null 가능
    ) { ... }
}
```

- `output`은 **추가가 필요하므로** `MutableList`
- `errors`는 **null 원소 가능성** 있으므로 `MutableList<String?>`

### 핵심 요약

- 자바에서 선언한 컬렉션은 코틀린에서 플랫폼 타입(`Type!`)으로 들어온다.
- 오버라이드나 사용 시 **널 가능성**, **원소의 null 허용 여부**, **변경 가능 여부**를 명확히 판단해야 한다.
- 플랫폼 타입의 해석은 **사용자의 책임**이다.
- 잘못된 해석은 런타임 오류나 설계 불일치로 이어질 수 있으니, 인터페이스 문서나 자바 코드의 의미를 정확히 파악하자.

## 배열과 성능, 상호 운용

### 코틀린 배열

| 생성 방식 | 설명 |
| --- | --- |
| `arrayOf("a", "b", "c")` | 지정한 원소로 배열 생성 |
| `arrayOfNulls<String>(3)` | null로 초기화된 지정 길이 배열 생성 |
| `Array(26) { i -> ... }` | 람다로 초기화된 배열 생성 |

```kotlin
val letters = Array(26) { i -> ('a' + i).toString() }
```

- `Array<T>`는 **객체 타입의 배열**을 표현함 (예: `Array<Int>` → `Integer[]` in Java)

### 컬렉션 → 배열 변환

- `toTypedArray()` 사용
- `vararg` 함수에 넘길 때는  스프레드 연산자 사용

```kotlin
val list = listOf("a", "b", "c")
println("%s/%s/%s".format(*list.toTypedArray()))
```

### 원시 타입 배열

| 타입 | 설명 |
| --- | --- |
| `IntArray` | 박싱 없는 `int[]` |
| `DoubleArray` | `double[]` |
| `BooleanArray`, `CharArray` 등 | 자바 원시 배열 대응 |
- 생성 방법

```kotlin
val arr = IntArray(5) // [0, 0, 0, 0, 0]
val arr2 = intArrayOf(1, 2, 3)
val squares = IntArray(5) { i -> (i + 1) * (i + 1) }
```

- `toIntArray()` 등으로 컬렉션에서 변환 가능

### 배열은 컬렉션과 같은 확장 함수 사용 가능

```kotlin
args.forEachIndexed { i, arg -> println("Argument $i: $arg") }
```

- `filter`, `map`, `forEach`, `joinToString` 등 모두 사용 가능 (결과는 리스트로 반환됨)

### 핵심 요약

- 코틀린 배열은 객체 타입을 위한 `Array<T>`와 원시 타입 최적화를 위한 `IntArray` 등으로 분리된다.
- `Array<T>`는 자바의 `T[]`, `IntArray`는 `int[]`로 컴파일됨.
- `Array`는 타입 안정성, `IntArray`는 성능을 보장.
- 자바와의 상호 운용성을 위해 배열이 필수일 때 `toTypedArray()` 등을 활용한다.

# 챕터 요약

### 🔢 기본 타입

- 코틀린의 `Int`, `Double` 등은 **JVM의 원시 타입**으로 컴파일됨 (`int`, `double`).
- `Int?`처럼 **널이 될 수 있는 원시 타입**은 **박싱된 타입**(`Integer`)으로 변환됨.
- **부호 없는 타입** (`UInt`, `UByte` 등)은 JVM에 없지만 **인라인 클래스**로 구현되어 **성능 손실 없이 사용 가능**.

---

### 🧬 특수 타입

- `Any`: 모든 타입의 최상위 타입. 자바의 `Object`와 대응.
- `Unit`: 반환값이 없는 함수에 사용. 자바의 `void`와 유사하지만 **값을 갖는 타입**.
- `Nothing`: **정상적으로 종료되지 않는 함수** (예: 예외 발생, 무한 루프 등)에 사용.

---

### 🔄 자바와의 상호 운용

- 자바 타입은 코틀린에서 **플랫폼 타입(String!)**으로 나타남 → 널 가능 여부를 **개발자가 결정**.
- 컬렉션도 마찬가지로 자바 타입은 읽기 전용/변경 가능 여부를 **명확히 지정**해야 함.
- 자바 코드에 컬렉션을 넘길 때는 null 여부, 변경 가능성에 대해 **명확히 판단할 책임이 개발자에게 있음**.

---

### 📚 컬렉션

- **읽기 전용(Collection)** vs **변경 가능(MutableCollection)** 인터페이스 구분이 중요.
- `filterNotNull()`, `map()` 등 함수형 API가 풍부하게 제공됨.
- 자바 컬렉션과 **동일한 클래스지만**, **인터페이스로 구분**해 사용 편의성을 높임.
- **불변 컬렉션은 별도 라이브러리(kotlinx.collections.immutable)** 사용 필요.

---

### 🧮 배열

- `Array<T>`는 **제네릭 배열**이고, 자바에서는 `T[]`로 컴파일됨.
- `IntArray`, `DoubleArray` 등은 **박싱 없이 원시 배열**(`int[]`)을 제공함.
- 배열 생성: `arrayOf(...)`, `Array(size) { ... }`, `intArrayOf(...)` 등
- **컬렉션 → 배열** 변환: `toTypedArray()`

# 오늘의 퀴즈

## 퀴즈 1번

다음 중 컴파일 오류 없이 작동하는 코드는?

```kotlin
fun printValue(x: Int) = println(x)

fun main() {
    val list = listOf(1L, 2L, 3L)
    val target = 1
    
    if (target in list) printValue(target)
}
```

A) 정상 동작

B) `target in list`에서 오류

C) `printValue(target)`에서 오류

D) `listOf(1L, 2L, 3L)`에서 오류


## 퀴즈 2번

다음 코드에서 `input`은 `null`일 수 있는 타입이지만, 안전한 호출 없이도 `input`을 바로 `return`에 사용할 수 있는 이유는?

```kotlin
fun check(input: String?): String {
    return input ?: fail("Input is required")
}

fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

**A.** `fail()` 함수는 예외를 던지므로 이후 코드는 실행되지 않는다.

**B.** `fail()` 함수의 반환 타입이 `Nothing`이므로, 컴파일러는 그 이후 코드가 없다고 간주하고 input이 절대 null이 아님을 추론한다.

**C.** 엘비스 연산자는 자동으로 null이 아닌 값만 전달하도록 보장한다.

**D.** 컴파일러는 런타임 예외 가능성을 감지하지 않기 때문에 nullable 값을 그대로 사용할 수 있다.


## 퀴즈 3번

다음 중 컴파일 오류가 발생하지 않는 코드는?

```kotlin
val items: List<String?>? = listOf("A", null, "B")
```

```kotlin
items.map { it.length } // A
items?.map { it.length } // B
items?.filterNotNull()?.map { it.length } // C
items.filterNotNull().map { it.length } // D
```

