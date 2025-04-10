## 코틀린은 식을 사랑한다

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

- 코틀린에서 if는 문이 아닌 식이다. (다른 언어에서의 삼항 연산자와 유사)
- `식`: 값을 만들어내고 하위 요소로 계산에 참여 가능
- `문`: 가장 안쪽 블록의 최상위 스코프. 아무런 값을 만들지 않음. (for, while, do/while 제외 대부분은 식임)

```kotlin
    val x = if (myBoolean) 3 else 5

    val direction = when (inputString) {
        "u" -> UP
        "d" -> DOWN
        else -> UNKNOWN
    }
    
    val number = try {
        inputString.toInt()
    } catch (e: NumberFormatException) {
        -1
    }
```

- if, when, try 모두 문이 아니라 식이다!
- 해당 스코프에서 만들어 계산해낸 값이 반환되고, 이를 변수에 할당할 수 있다.

<aside>
📝

여기서 뇌가 살짝 녹을 뻔했다… 

내가 경험한 모든 언어에서는 문으로 취급받던 애들이 여기서는 식이다.

코틀린에서는 습관적으로 if 문, switch 문이라고 부르면 혼난다.

</aside>

```kotlin
val number: Int
val alsoNumber = i = getNumber()

// ERROR: Assignments are not expressions,
// If and only expressions are allowed in this context
```

- 대입은 반대로 식이 아니라 문으로 취급된다.

## 블록 본문 함수 vs 식 본문 함수

```kotlin
// 블록 본문 함수
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}

// 식 본문 함수
fun max(a: Int, b: Int): Int = if (a > b) a else b

// 식 본문 함수 (반환 타입 생략)
fun max(a: Int, b: Int) = if (a > b) a else b
```

- 코틀린에서는 식 본문 함수를 더 많이 사용한다.
- 식 본문 함수에서는 본문 식을 분석해서 함수 반환 타입을 추론해준다.
- 때문에 반환 타입을 생략할 수 있다.

<aside>
📝

그래도 라이브러리를 작성할 때는 반환 타입을 명시하는 것이 좋다고 한다.

</aside>

## val vs var

### val

- 값을 뜻하는 value 에서 따왔다.
- read-only 변수이다. 한 번만 대입 가능하다. (자바의 final과 동등)

### var

- 변수를 뜻하는 variable 에서 따왔다.
- 재할당 가능한 변수이다. (여러 번 대입 가능) (자바의 일반 변수와 동등)

<aside>
📝

코틀린은 가능한 모든 변수에 val을 사용하는 것을 지향한다.

</aside>

- 한 번 지정된 타입은 변환 함수를 통해 강제 형 변환만 가능하다.

## 문자열 템플릿

```kotlin
fun main() {
    val input = readln()
    val name = if (input.isNotBlank()) input else "Kotlin"
    println("Hello, $name!")
}
```

- `$변수명`을 통해 문자열 템플릿 기능을 지원한다.

```kotlin
fun main() {
    val input = readln()
    val name = if (input.isNotBlank()) input else "Kotlin"
    println("$name님 반가워요.")
}
```

- 코틀린에서는 모든 유니코드 문자를 변수로 사용할 수 있어, 한글도 변수로 선언가능하다.
- 따라서 다음은 `name님` 이라는 변수를 선언한 것과 같다..!

```kotlin
fun main() {
    val input = readln()
    val name = if (input.isNotBlank()) input else "Kotlin"
    println("${name}님 반가워요.")
}
```

- `${변수명}` 같은 형태도 지원하니, 습관적으로 이렇게 사용하는 것이 안전할 것 같다.

```kotlin
fun main() {
    val name = readln()
    println("Hello, ${if (name.isBlank()) "someone" else name}!")
}
```

- 식을 문자열 템플릿에 포함시킬 수 있다.
- if는 식이다.
- 따라서 위와 같은 코드도 가능하다.

## 코틀린에서의 클래스

```kotlin
class Person(val name: String)
```

- 간결성이 핵심
- public이 default
- getter 자동 지원
- 자바의 record와 유사한 개념

```kotlin
fun main() {
    var person = Person("John Doe", true)
    
    person.name = "Jane Doe" // Error: Val cannot be reassigned
    person.isStudent = false // This is allowed since isStudent is a var
}

class Person(val name: String, var isStudent: Boolean)
```

- val가 기본값
- val 을 쓰면 읽기전용, var를 쓰면 가변 프로퍼티
- val로 선언한 변수는 getter만 제공
- var로 선언한 변수는 getter, setter 제공

## 커스텀 접근자

```kotlin
class Rectangle(val width: Int, val height: Int) {
    val isSquare: Boolean
        get() = width == height
}
```

- 객체 안의 다른 프로퍼티로 계산이 되는 경우 → 커스텀 접근자 구현 필요
- 접근할 때 계산한다고 해서 `온 더 고` 프로퍼티라고 부름

<aside>
📝

식 본문 함수를 써주는게 코틀린스러울 것 같다.

</aside>

## 자바 vs 코틀린 - 패키지 구조의 차이

![image.png](attachment:8a470a7e-93cf-4782-a917-19a43b64218e:image.png)

![image.png](attachment:1a6db6b2-f4f3-435d-b77b-f8314410398b:image.png)

<aside>
📝

그렇지만 자바와 마찬가지로 패키지별로 디렉토리를 구성하는 것이 좋다고 한다.

하지만 여러 클래스를 한 파일에 넣는 것은 권장한다고 한다. 특히 소스코드의 크기가 작을수록.

</aside>

## 코틀린의 enum

- enum은 소프트 키워드이다.
- `소프트 키워드`: class 앞에 있을 때는 특별한 의미. 그 외에 곳에는 일반적 의미
- `하드 키워드`: class는 식별자로 사용 불가, 클래스를 표현하는 변수 등은 clazz 등으로 표현

```kotlin
fun main() {
    println(Color.CYAN.rgb)
}

enum class Color(
    val r: Int,
    val g: Int,
    val b: Int,
) {
    RED(255, 0, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255),
    YELLOW(255, 255, 0),
    CYAN(0, 255, 255),
    MAGENTA(255, 0, 255);

    val rgb: Int
        get() = (r * 256 + g) * 256 + b
    fun printColor() = println("Color: $name, RGB: $rgb")
}
```

- enum에서도 커스텀 접근자가 요긴하게 쓰일 수 있는 것 같다.

## when과 enum의 만남

```kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "RICHARD"
        Color.GREEN -> "GAVE"
        Color.BLUE -> "BATTLE"
        Color.YELLOW -> "YORK"
        Color.CYAN -> "CLIMBED"
        Color.MAGENTA -> "MOUNTAINS"
    }
```

- enum과 when을 함께 쓸 때는 분기 끝에 else를 넣지 않아도 된다.
- 컴파일러는 when이 `철저한지` 검사한다.
- 해당 함수에서 enum이 모든 가능한 분기에서 값을 만들어내므로 철저하다고 판단되어, else를 넣지 않아도 되는 것이다.
- 모든 분기에서 값을 만들어낼 수 없다고 판단되면, else를 넣어야만 한다.
- when도 식이므로 식 본문 함수로 응용할 수 있다.

```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
        setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
        setOf(Color.BLUE, Color.CYAN) -> Color.BLUE
        setOf(Color.CYAN, Color.MAGENTA) -> Color.MAGENTA
        else -> throw Exception("Dirty color")
    }
```

- 인자들을 포함하는 Set 객체를 만드는 `setOf` 라는 함수가 존재한다.
- 이를 응용하여 다음과 같은 처리를 할 수 있다.
- 모든 분기 식에서 만족하는 조건을 찾을 수 없으면 else 분기를 계산하게 된다.

## 인자 없는 when

```kotlin
fun mixOptimized(c1: Color, c2: Color) =
    when {
        (c1 == Color.RED && c2 == Color.YELLOW) ||
                (c1 == Color.YELLOW && c2 == Color.RED) -> Color.ORANGE
        (c1 == Color.YELLOW && c2 == Color.BLUE) ||
                (c1 == Color.BLUE && c2 == Color.YELLOW) -> Color.GREEN
        (c1 == Color.BLUE && c2 == Color.CYAN) ||
                (c1 == Color.CYAN && c2 == Color.BLUE) -> Color.BLUE
        (c1 == Color.CYAN && c2 == Color.MAGENTA) ||
                (c1 == Color.MAGENTA && c2 == Color.CYAN) -> Color.MAGENTA
        else -> throw Exception("Dirty color")
    }
```

- 아무 인자를 넣지 않고 이런식으로 최적화도 가능하다.
- 코드 가독성 vs 성능 사이의 trade-off를 고려해볼 수 있다.

## 스마트 캐스트

```kotlin
fun main() {
    println(eval(Sum(Num(1), Num(2)))) // 3
}

interface Expr // 마크업 인터페이스
class Num(val value: Int) : Expr // 리프 노드
class Sum(val left: Expr, val right: Expr) : Expr // 합 노드

fun eval(e: Expr): Int {
    if (e is Num) {
        val n = e as Num // 불필요한 캐스트
        return n.value
    }
    if (e is Sum) {
        return eval(e.left) + eval(e.right) // 변수 e에 대해 스마트 캐스트
    }
    throw IllegalArgumentException("Unknown expression")
}
```

- 코틀린의 is는 단순히 타입 검사 뿐만 아니라 스마트 캐스트를 지원한다.
- 타입 검사한 변수를 마치 그 타입의 변수인 것 처럼 사용할 수 있다.
- 스마트 캐스트를 사용한다면 그 프러퍼티는 반드시 val이어야 한다.
- 커스텀 접근자를 사용한 것이어도 안된다. (항상 같은 값을 내놓는다는 확신 불가)

## 연쇄 if 대신 when 활용

```kotlin
fun eval(e: Expr): Int {
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.left) + eval(e.right)
        else -> throw IllegalArgumentException("Unknown expression")
    }
}
```

- 연쇄 if 식이 발생하는 부분에 when을 적용해보자.

```kotlin
fun eval(e: Expr): Int =
    when (e) {
        is Num -> {
            println("Num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left = eval(e.left)
            val right = eval(e.right)
            println("Sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

- when 식 안에는 블록 스코프를 넣을 수 있다.
- 가장 마지막 문장이 블록 전체의 반환값이 된다.
- 이는 try-catch 식에서도 마찬가지로 적용된다.

## 이터레이션

```kotlin
fun main() {
    for (i in 1..100) {
        println(삼육구(i))
    }
    
    for (i in 100 downTo 1 step 2) {
        println(삼육구(i))
    }
}

fun 삼육구(i: Int) = when {
    i % 15 == 0 -> "짝짝"
    i % 5 == 0 -> "짝"
    i % 3 == 0 -> "삼"
    else -> "$i"
}
```

- 파이썬의 for in range(1, 101)과 유사하게 느껴졌다.
- `downTo`를 활용하여 구육삼도 즐길 수 있다. (역방향 순열)
- `step`을 통해 증감분 설정도 가능하다.

```kotlin
fun main() {
    val binaryReps = mutableMapOf<Char, String>()
    for (char in 'A'..'Z') {
        val binary = char.code.toString(radix = 2) // Convert to binary
        binaryReps[char] = binary
    }

    for ((letter, binary) in binaryReps) {
        println("$letter: $binary")
    }
}
```

- 단순히 코틀린에서 맵 선언 후 이터레이션 하는 방법이다.
- 맵을 해체하여 별도의 변수에 저장할 수 있다.

```kotlin
fun main() {
    println(isLetter('a')) // true
    println(isNotDigit('5')) // true
}
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

```

- `in` 키워드를 사용하여 범위에 속하는지 검사 가능하다.
- 안 속하는지 보려면 `!in`을 쓰면 된다.

## 코틀린에서의 Exception

```kotlin
fun main() {
    val number = 101 // Example number to validate
    val percentage =
        if (number in 0..100)
            number
        else
            throw IllegalArgumentException(
                "A percentage must be between 0 and 100: $number"
            )
    println("Valid percentage: $percentage")
}
```

<aside>
📝

코틀린에서는 new 키워드가 없다.

예외 인스턴스를 만들 때도 예외는 없다.

</aside>

## try, catch, finally 사용법

```kotlin
fun main() {
    val reader = BufferedReader(StringReader("a"))
    println(readNumber(reader)) // Should print 123
}

fun readNumber(reader: BufferedReader): Int? {
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        println("Invalid number format: ${e.message}")
        return null
    }
    finally {
        reader.close()
    }
}
```

- throws 절이 코틀린에는 없다.
- 자바에서는 Checked Exception 에 대해 필수로 붙여야 한다.
- 코틀린에서는 Checked Exception과 UnChecked Exception를 구분하지 않는다.

## 식으로 try 사용하기

```kotlin
fun main() {
    val reader = BufferedReader(StringReader("a"))
    readNumber(reader) // 아무것도 출력되지 않음
}

fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        return
    }
    println("Parsed number: $number")
}

```

- try는 반드시 중괄호로 사용해야 한다.
- 식으로 사용할 때도 마찬가지이다.
- 여기서는 catch 에는 return 문을 사용하였기 때문에 다음 코드가 실행되지 않는다. (문으로써 사용된 것이다.)

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        null
    }
    println("Parsed number: $number")
}
```

- catch 문 대신 catch 식을 사용하고 싶다면, 식을 넣으면 된다!
- null 값을 명시하여, null 값이 반환된다.

## 코린이의 다섯줄평

- 코틀린은 식을 좋아한다.
- 코틀린의 식 사랑에 뇌가 살짝 녹을 뻔했다. (if 식, when 식, 식 본문 함수 등)
- 편의 함수를 만들기 굉장히 편의한 언어인 것 같다.
- 왜 파일 안에 여러 클래스를 선언하는 것을 주저하지 말라고 하는지 알 것 같다.
- 모든 개발자가 코틀린에 최적화된 뇌를 가지고 있다면, 세상이 많이 평화로워질 것 같다.

## 작가의 요약

- **함수**를 정의할 때는 `fun` 키워드를 사용한다.
- `val`과 `var`는 각각 **읽기 전용 변수**와 **변경 가능한 변수**를 선언할 때 쓰인다.
- `val` 참조는 읽기 전용이지만, `val`이 가리키는 객체의 **내부 상태는 여전히 변경 가능**할 수 있다.
- 문자열 템플릿을 사용하면 지저분하게 문자열을 연결하지 않아도 된다. 변수 이름 앞에 `$`를 붙이거나 식을 `${식}`처럼 `${}`로 둘러싸면 변수나 식의 값을 문자열 안에 넣을 수 있다.
- 코틀린에서는 **클래스를 아주 간결하게 표현**할 수 있다.
- 다른 언어에도 있는 `if`는 코틀린에서는 **식(expression)**이며, 값을 만들어낸다.
- 코틀린의 `when`은 자바의 `switch`와 비슷하지만 **더 강력**하다.
- 어떤 변수의 타입을 검사하고 나면, 굳이 그 변수를 캐스팅하지 않아도 검사한 타입의 변수처럼 사용할 수 있다. **컴파일러가 스마트 캐스트**를 활용해 자동으로 타입을 바꿔준다.
- `for`, `while`, `do-while` 루프는 자바에서 제공하는 동일한 키워드의 기능과 비슷하다. 하지만 코틀린의 `for`는 자바 `for`보다 더 편리하다. 특히 **맵을 이터레이션하거나, 인덱스를 사용해 컬렉션을 순회할 때** 코틀린의 `for`가 더 편리하다.
- `1..5` (그리고 `1..<5`)와 같은 식은 **범위(range)**를 만들어낸다. 범위와 순열은 코틀린에서 같은 문법과 추상화를 `for` 루프에 사용할 수 있게 해주고, 어떤 값이 범위 안에 들어있거나 들어있지 않은지 검사하기 위해 `in`이나 `!in`과 함께 사용할 수 있다.
- 코틀린의 예외 처리는 자바와 아주 비슷하다. 다만 코틀린에서는 함수가 던질 수 있는 예외를 **명시적으로 선언하지 않아도 된다.**
