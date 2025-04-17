# 코틀린의 컬렉션

```kotlin
fun main() {
    val set = setOf(1, 2, 3, 4, 5)
    val list = listOf(1, 2, 3, 4, 5)
    val map = mapOf(1 to "one", 2 to "two", 3 to "three", 4 to "four", 5 to "five")

    println(set.javaClass) // (set as java.lang.Object).getClass() 와 동등
    println(list.javaClass)
    println(map.javaClass)
}

class java.util.LinkedHashSet
class java.util.Arrays$ArrayList
class java.util.LinkedHashMap
```

- 코틀린의 컬렉션은 자바 컬렉션 클래스를 래핑하여 사용한다.
- 코틀린 컬렉션 인터페이스는 default가 읽기 전용이다.

```kotlin
fun main() {
    val strings = listOf("apple", "banana", "cherry")

    println(strings.last()) // Output: cherry
    println(strings.shuffled()) // Output: Randomly shuffled list

    val numbers = listOf(1, 2, 3, 4, 5)
    println(numbers.sum()) // Output: 15
}
```

- 자바 Collection 클래스를 래핑하여 추가적인 기능을 제공한다.
- last, shuffled, sum 모두 코틀린에서만 제공되는 기능이다.

# 코틀린 함수의 인자

대부분의 언어에서 지원되는 방식과 유사하며, 상식선에서 동작한다.

## Named Arguments

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4, 5)
    println(joinToString(collection = numbers, separator = " ", prefix = " ", postfix = " "))
}
```

- 코틀린에서는 자바와 달리 함수에 전달하는 인자의 이름을 지정할 수 있다.
- 모든 인자를 쓸 수도 일부를 생략할 수도 있다.

## Default Parameter

```kotlin
fun main() {
    val numbers = listOf(1, 2, 3, 4, 5)
    println(joinToString(collection = numbers))
}

fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = " ",
    postfix: String = " ",
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

- 코틀린에서는 default 인자값을 지정할 수 있다.
- 모든 인자를 쓸 수도 일부를 생략할 수도 있다.

# 클래스 밖에서 선언하는 코틀린 함수

- 자바에서는 어떤 경우에 특별한 상태나 인스턴스 메소드가 없는 클래스가 생겨나는 경우가 많다.
- 보통 이런 클래스들은 정적 메소드들의 모음집 역할을 담당한다.

<aside>
📝

여러분이 작성한 코드에서 비슷한예를 보고싶다면 Util이 이름에 들어있는 클래스를 찾으면 된다.

이 문구가 너무 웃겼다 ,,,

실제로 까비 레포에서 [DateUtil.java](http://DateUtil.java) 를 검색하면, `13개 짜리 정적 메소드들을 모아둔 클래스`를 만날 수 있다.

</aside>

- JVM에서는 클래스 안에 들어있는 코드만 실행가능하다.
- 코틀린은 이를 위해 컴파일 과정에서 클래스 밖에 선언된 메소드들을 파일 이름에 해당하는 클래스로 생성한다.
- 때문에 코틀린에서 작성한 `클래스 밖에 선언된 메소드`들도 `자바에서 호출 가능`하다.
- `@file:JvmName("StringFunctions")` 파일에 대응하는 클래스 명을 바꾸고 싶다면, 패키지 이름 선언 앞에 해당 어노테이션을 붙이면 된다고 한다.

# 클래스 밖에서 선언하는 코틀린 프로퍼티(상수)

```kotlin

// 코틀린
public const val UNIX_LINE_SEPARATOR = "\n"

// 자바 (클래스 내에 선언)
public static final String UNIX_LINE_SEPARATOR = "\n"
```

- 함수와 마찬가지로 프로퍼티도 파일 최상위에 선언할 수 있다.
- 자바에서 클래스 내에 선언하는 함수와 동등하다.

```kotlin
@SinceKotlin("1.2")
public const val PI: Double = 3.141592653589793

@SinceKotlin("1.2")
@InlineOnly
public actual inline fun max(a: Int, b: Int): Int = nativeMath.max(a, b)
```

- `kotlin.math` 패키지에는 `PI`와 `max`가 각각 최상위 프로퍼티와 함수로 제공된다.

# 확장 함수

```kotlin
fun main() {
    println("Kotlin".lastChar()) // n
}

// String 클래스를 확장하여, lastChar를 추가한 것 처럼 보이기 한다.
fun String.lastChar(): Char = this.get(this.length - 1)

// this는 생략가능
fun String.lastChar(): Char = this.get(length - 1)
```

- 어떤 클래스의 멤버 메소드인 것 처럼 호출할 수 있지만, 실제로는 그 클래스 밖에 선언된 함수를 말한다.
- `수신 객체 타입`: String
- `수식 객체`: this
- 확장 함수는 자바나 코틀린 뿐만 아니라 groovy같이 JVM 기반 언어라면 모두 가능하다. (심지어 final로 선언되어 있어도 가능하다.)

<aside>
📝

final도 확장가능하면 final이 아닌거 아닌가..?

</aside>

- 확장 함수에서 private나 protected 멤버를 참조할 수는 없어, 캡슐화를 깨지는 않는다.

## 확장 함수 import

```kotlin
import strings.lastChar // strings 패키지에 정의했다고 가정

val c = "Kotlin".lastChar()

import strings.* // 와일드카드 허용

val c = "Kotlin".lastChar()

import strings.lastChar as last // Python이나 JS처럼 import 할 때 as 키워드 가능
val c = "Kotlin".last()

// 자바에서는 이렇게 변환된다.
char c = StringUtilKt.lastChar("Java");
```

- 확장 함수를 쓰려면 해당 확장 함수가 선언된 패키지를 import 해야 한다. (충돌 방지를 위해)
- 여러 패키지에 이름이 동일한 함수가 많을 때 as 키워드를 사용한다.

## 확장 함수 활용

```kotlin
fun main() {
    val list = listOf(1, 2, 3)
    println(list.joinToString(separator = ", "))
}

fun <T> Collection<T>.joinToString( // Collection<T> == 수신 객체 타입
    separator: String = ", ",
    prefix: String = " ",
    postfix: String = " ",
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) { // this == 수신 객체
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

## 확장 함수는 오버라이드 불가

### **일반적인 멤버 함수 오버라이드**

```kotlin
fun main() {
    val view: View = Button()
    view.click() // Button Clicked
}

open class View {
    open fun click() = println("View Clicked")
}

class Button: View() {
    override fun click() = println("Button Clicked")
}
```

### **확장 함수 오버라이드 (불가)**

```kotlin
fun View.showOff() = println("You are fool view 😢")
fun Button.showOff() = println("You are fool button 😢")

fun main() {
    val view: View = Button()
    view.showOff() // You are fool view 😢
}

open class View {
    open fun click() = println("View Clicked")
}

class Button: View() {
    override fun click() = println("Button Clicked")
}
```

- 파라미터가 완전히 같은 확장 함수를 정의할 수는 있다.
- 확장 함수의 변환은 컴파일 타임에 이루어진다.
- 때문에 실제 호출될 함수는 컴파일 타임에 타입에 결정되고, 런타임에 그 변수에 저장된 객체 타입에 의해 결정되지 않는다.

```kotlin
class Demo {
	public static void main(String[] args) {
		View view = new Button();
		ExtensionKt.showoff(view); // You are fool view 😢
	}
}
```

- 자바로 치면 이렇게 클래스에 정적 메소드를 정의해둔 showoff 함수를 호출하는 것과 동등하다.

## 확장 프로퍼티

- 확장 프로퍼티는 프로퍼티라는 이름을 쓰지만 자바 객체 인스턴스에 필드를 실제로 추가하지는 않는다. 
즉, 아무런 상태를 가질 수 없다.
- 그 말은 즉, 확장 프로퍼티는 커스텀 접근자 정의를 통해 구현할 수 있다.

```kotlin
fun main() {
    val sb = StringBuilder("sichoi?")
    println(sb.lastChar) // ?

    sb.lastChar = '!'
    println(sb.lastChar) // !
}

var StringBuilder.lastChar: Char
    get() = get(length - 1) // 프로퍼티의 getter
    set(value: Char) { // 프로퍼티의 setter
        this.setCharAt(length - 1, value)
    }
```

- 자바에서는 다음과 같이 확장 프로퍼티를 사용할 수 있다.
    - `StringUtilKt.getLastChar(”sichoi?”)`
    - `StringUtilKt.getLastChar(sb, ”?”)`

# 코틀린 컬렉션의 비밀

## 수 많은 확장 함수를 제공

```kotlin
fun main() {
    val strings: List<String> = listOf("first", "second", "third")
    println(strings.last()) // third

    val numbers: Collection<Int> = setOf(1, 2, 3)
    println(numbers.sum()) // 6
}

public fun <T> List<T>.last(): T {
    if (isEmpty())
        throw NoSuchElementException("List is empty.")
    return this[lastIndex]
}

public fun Iterable<Byte>.sum(): Int {
    var sum: Int = 0
    for (element in this) {
        sum += element
    }
    return sum
}
```

- List와 Collection의 last와 sum은 모두 확장 함수로 정의된다.
- 이외에 많은 코틀린 표준 라이브러리에서는 확장 함수를 포함하여, 여러가지 편의 기능을 제공한다.

## 가변 인자 함수

```kotlin
fun main() {
    val list = listOf(1, 2, 3, 4, 5)
    println(list)
}

public fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()

```

- 자바에서는 가변 길이 인자를 사용할 때 타입 뒤에 … 을 사용한다.
- 코틀린에서는 명시적으로 vararg 라는 변경자를 붙인다.
- 자바에서는 가변 인자 필드에 배열을 통째로 넘기면 된다.
- 코틀린에서는 배열을 명시적으로 spread 하여 전달해야 한다.

<aside>
📝

스프레드 연산자는 * 이다.

</aside>

## 튜플을 다루는 방법

```kotlin
fun main() {
    val map = mapOf(1 to "one", 2 to "two", 3 to "three")
    println(map)

}
```

- 사난님이 스포(?)하신 것처럼 to는 키워드가 아니다.
- `중위 호출 방식`을 활용하는 일반 메소드이다.

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```

- 인자가 하나뿐인 일반 메소드나 확장 함수에 대해서는 중위 호출이 가능하다.
- 함수에 중위 호출을 허용하고 싶다면, infix 변경자를 추가하면 된다.
- to 함수는 Pair 인스턴스를 반환한다.

```kotlin
val (number, name) = 1 to "one"
```

- 이런식으로 Pair 객체를 분해하여 각각의 변수에 할당할 수도 있다.
- 이런 기능을 `구조 분해 선언`이라고 부른다.

# 오늘의 문제

## 1번 문제

```kotlin
open class Greeter {
    fun greet() = "Hello from Greeter"
}

// 확장 함수 정의
fun Greeter.greet() = "Hello from Extension"

fun main() {
    val greeter = Greeter()
    println(greeter.greet())
}
```

- 이 코드의 출력 결과는 ?

## 2번 문제

```kotlin
val StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }

fun main() {
    val sb = StringBuilder("Hello")
    sb.lastChar = '!'
    println(sb.toString())
}

```

- 이 코드의 문제점은 ?
