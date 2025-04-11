# 1. 기본 요소: 함수와 변수
- 함수 선언 시 fun 키워드 사용
- 함수를 클래스 안에 넣어야 할 필요가 없다.
- 코틀린 표준 라이브러리는 표준 자바 라이브러리 함수에 대해 더 간결한 구문을 사용할 수 있도록 wrapper제공(ex. println)
- 줄 끝에 세미콜론 붙이지 않아도 됨.
## 파라미터와 반환값이 있는 함수 선언
- 파라미터 이름이 먼저 오고 그 뒤에 타입 지정(콜론(:)으로 구분)
> 코틀린에서 루프를 제외한 대부분의 제어구조가 식. 다른 식의 하위 요소로 계산에 참여 가능 <br/>
> 코틀린에서 대입은 항상 문으로 취급. `val alsoNumber = i = getNumber()` - 오류
## 식 본문 함수 <-> 블록 본문 함수
- 함수 본문이 식 하나로 이루어져있으면 중괄호를 없애고 return을 제거하여 **식 본문 함수**로 변경 가능
```kotlin
fun max(a: Int, b: Int): Int {
  return if (a>b) a else b
}
```
- 식 본문 함수의 리턴 타입은 생략 가능.
- 블록 본문 함수는 반환 타입 지정하고 return문을 사용해 반환값 명시.
## 변수 선언
- val : 읽기 전용 참조. 초기화하고 나면 다른 값 대입 X
- var : 재대입 가능한 참조. 초기화 이후에 다른 값 대입 O
- 기본적으로 코틀린에서 모든 변수를 val 키워드를 사용해 선언, 반드시 필요할 때에만 var로 변경.
- val 참조 자체의 값을 바꿀 수는 없어도 참조가 가리키는 객체의 내부 값은 변경될 수 있다.
  (ex. val가 가리키는 가변 리스트에 원소 추가 가능)
```kotlin
fun main() {
  val languages = mutableListOf("Java")
  laguages.add("Kotlin")
}
```
- var 키워드 사용 시 변수의 값을 변경할 수 있지만 타입은 고정됨.

## 문자열 템플릿
```kotlin
fun main() {
  val name = readln()
  if (name.isNotBlank()) {
    println("Hello, ${name.length}-letter person!")
  }
}
```
- 한글을 문자열 템플릿에서 사용할 경우 변수명을 {}로 감싸서 확실히 구분하기
# 2. 행동과 데이터 캡슐화: 클래스와 프로퍼티
- 코틀린은 클래스를 간결하게 정의할 수 있는 문법 제공.
- name이라는 속성을 가지고 있는 Person클래스 정의
  -> `class Person(val name: String)`
## 프로퍼티
- 프로퍼티: 필드와 접근자를 한데 묶음
- 코틀린은 프로퍼티를 언어 기본 기능으로 제공. 자바의 필드와 접근자 메서드를 완전히 대신함.
- 값을 저장하기 위한 비공개 필드와 변수의 종류에 따라 게터, 세터를 만듬.
- 코틀린의 게터, 세터 명명 규칙
  - 게터는 프로퍼티명 앞에 get을 붙인다. 단, 이름이 is로 시작하는 경우 원래이름을 그대로 사용한다.
  - 세터는 이름이 is로 시작하는 경우 is를 set으로 바꾼 이름을 사용한다.
    - isStudent() -> ???
- ***게터/세터를 명시적으로 호출하는 대신 프로퍼티 이름을 직접 사용해도 코틀린이 알아서 게터/세터 호출해줌.***
## 커스텀 접근자
- 프로퍼티에 접근할 때 계산을 할 수 있는 'on the go'프로퍼티라면 별도의 필드에 저장할 필요 없이 프로퍼티에 접근할 때마다
  계산하여 반환 가능
  ```kotlin
  class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
      get() = height == width // getter 선언
  }
  ```
## 코틀린 소스코드 구조: 디렉터리와 패키지
- 모든 코틀린 파일에는 맨 앞에 package문이 올 수 있음. package문이 있는 파일에 들어있는 모든 선언을 해당 패키지 안에 들어감.
- 다른 패키지에 정의한 선언을 사용하려면 import 키워드를 이용해 불러와야 함.
- 자바에서는 패키지의 구조와 일치하는 디렉터리 계층 구조를 만들어야 하지만 코틀린은 X(그렇지만 대부분 따르는게 낫다)
# 3. 선택 표현과 처리: enum과 when
## enum 클래스와 enum 상수 정의
> enum은 소프트 키워드 : enum class로 사용하지 않는 경우 일반적인 이름으로 사용가능 <-> class는 하드키워드, 식별자로 사용 불가
- enum 클래스 안에 메서드를 정의하는 경우 **반드시** 이넘 상수 목록과 메서드 정의 사이에 세미콜론 넣어야 함.
```kotlin
package ch02.colors

enum class Color(
  val r: Int,
  val g: Int,
  val b: Int
) {
  RED(255, 0, 0),
  ORANGE(255, 165, 0),
  YELLOW(255, 255, 0),
  GREEN(0, 255, 0),
  BLUE(0, 0, 255),
  INDIGO(75, 0, 130),
  VIOLET(238, 130, 238);

  fun rgb() = (r * 256 + g) * 256 + b
  fun printColor() = println("$this is $rgb")
}

fun main() {
  println(Color.BLUE.rgb)
  // 255
  Color.GREEN.printColor()
  // GREEN is 65280
}
```
## when
- 컴파일러가 모든 가능한 경로를 처리할 수 없다고 판단하면 디폴트케이스 추가해야 함(else)
### 인자 없는 when의 사용
- 불필요한 객체 생성 막기 위해 사용 가능
- 가독성은 조금 떨어지지만 성능 향상 가능
- 인자가 없으려면 각 분기의 조건이 boolean결과를 계산하는 식이어야 함.
## 스마트캐스트
- 마커 인터페이스 : 여러 타입의 식 객체를 아우르는 공통 타입 역할을 수행하는 인터페이스
- is : 코틀린의 is 검사는 어떤 변수의 타입을 확인 후 그 타입에 속한 멤버에 접근하기 위해 변수 타입을 변환할 필요 없음.
  컴파일러가 대신 변환(스마트캐스트)
- 스마트 캐스트는 is로 변수의 타입을 검사한 다음에 값이 바뀔 수 없는 경우에만 작동.
  <br/> ex. 클래스의 프로퍼티는 val이어야 하고, 커스텀 접근자도 사용 X

# 4. 대상 이터레이션: while과 for 루프
- 코틀린에서는 레이블(@)을 지정하여 지정한 루프를 빠져나가거나 할 수 있음.
  ```kotlin
  outer@ while(outerCondition) {
    while (innerCondition) {
      if (shouldExit) break@outer
      if (shouldSkip) continue@outer
    }
  }
  ```
- 코틀린에서 범위를 쓸 때는 .. 연산자 사용(폐구간)
- 끝 값을 포함하지 않고 싶으면 ..< 연산자 사용
- `for (i in 100 downTo 1 step 2) {}`
- 인덱스와 함께 이터레이션(withIndex())
  ```kotlin
  fun main() {
    val list = listOf("10", "11", "1001")
    for ((index, element) in list.withIndex()) {
      println("$index: $element")
    }
    // 0: 10
    // 1: 11
    // 2: 1001
  }
  ```
- in으로 컬렉션이나 범위를 돌 때도 사용하는 것 외에도 범위 내에 어떤 값이 들어있는지 확인할 때에도 사용
# 코틀린에서 예외 던지고 잡아내기
- 자바와 달리 코틀린의 throw는 식이므로 다른 식에 포함 가능.
- 코틀린은 throws가 없음.(체크 예외에 대해서 명시 X)


# 문제
### 1. 다음 코드를 식 본문 함수로 바꿔보자.
```kotlin
fun max(a: Int, b: Int): Int {
  return if (a>b) a else b
}
```

### 2. 다음 코드를 좀더 코틀린 답게 바꿔보자.
```kotlin
fun eval(e: Expr): Int {
  if (e is Num) {
    val n = e as Num
    return n.value
  } else if (e is Sum) {
    return eval(e.right) + eval(e.left)
  } else {
    throw IllegalArgumentException("Unknown expression")
  }

fun main() {
  println(eval(Sum(Num(1), Num(2))))
}
```

### 1. 답
```kotlin
fun max(a: Int, b: Int) = if (a>b) a else b
```

### 2. 답
```kotlin
fun eval(e: Expr): Int =
  when (e) {
    is Num -> e.value
    is Sum -> eval(e.right) + eval(e.left)
    else -> throw IllegalArgumentException("Unknown expression")
  }
```


  
