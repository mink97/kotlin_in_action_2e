> 5장에서 다루는 내용
> - 람다식과 멤버 참조를 사용해 코드 조각과 행동 방식을 함수에게 전달
> - 코틀린에서 함수형 인터페이스를 정의하고 자바의 함수형 인터페이스 사용
> - 수신 객체 지정 람다 사용
---

- 람다식(람다) : 다른 함수에 넘길 수 있는 작은 코드 조각
  - 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아낼 수 있음.

# 1. 람다식과 멤버 참조
##  람다 소개: 코드 불록을 값으로 다루기
- 클래스를 선언하고 그 클래스의 인스턴스를 함수에 넘기는 대신, 함수를 직접 다른 함수에 전달
- 람다식을 사용하면 코드가 더욱 간결해진다.
- 함수를 선언할 필요없이 코드 블록을 직접 함수의 인자로 전달 가능
> 함수형 프로그래밍의 특성
> 1. **일급 시민인 함수**: 함수를 값으로 다룰 수 있다.
> 2. **불변성**: 객체를 만들 때 일단 만들어진 다음에는 내부 상태가 변하지 않음을 보장하는 방법으로 설계할 수 있다.
> 3. **부수 효과 없음**: 함수가 항상 똑같은 입력에 대해 같은 출력을 내놓는다. 함수가 다른 객체나 외부세계의 상태를 변경하지 않는다.
- 리스너 구현하기
  1. object 선언
    ```kotlin
    button.setOnClickListener(object: OnClickListener {
      override fun onClick(v: View) {
        println("I was clicked!")
      }
    })
    ```
  2. 람다
     ```kotlin
     button.setOnClickListener {
       println("I was clicked!")
     }
     ```
     ![image](https://github.com/user-attachments/assets/e3f699f3-84f9-415e-9515-393515acf479)
---
## 람다와 컬렉션
- Person클래스 내 가장 연장자 찾기
  1. for 루프로 직접 검색
  ```kotlin
  data class Person(val name: String, val age: Int)
  
  fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null
    for (person in people) {
      if (person.age > maxAge) {
        maxAge = person.age
        theOldest = person
      }
    }
    println(theOldest)
  }

  fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    findTheOldest(people)
  }
  ```
  2. maxByOrNull 함수 사용
  ```kotlin
  data class Person(val name: String, val age: Int)
  
  fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.maxByOrNull { it.age } )
  }
  ```
- 모든 컬렉션에 대해 maxByOrNull함수 호출 가능(인자 하나 받음)
- 람다가 인자를 하나만 받고 그 인자에 구체적 이름을 붙이고 싶지 않기 때문에 **it**이라는 암시적 이름 사용
- 람다가 단순히 함수나 프로퍼티에 위임할 경우에는 멤버 참조를 사용할 수 있음(??)
  `people.maxByOrNull(Person::age)`
- *컬렉션에 대해 수행하는 대부분의 작업은 람다나 멤버 참조를 인자로 취하는 라이브러리 함수를 통해 간결하게 표현할 수 있다.*
---
## 람다식의 문법
- 람다를 따로 선언해서 변수에 저장가능. But, 함수에 인자로 넘기면서 바로 람다를 정의하는 경우가 대부분
![image](https://github.com/user-attachments/assets/783696c4-9f2f-4675-99a9-5566bcabb92b)
- 코틀린 람다식은 항상 중괄호로 둘러싸여 있음.
- **인자 목록 주변에 괄호가 없다**(화살표가 인자 목록과 람다 본문을 구분)
### 1. 람다식 변수에 저장
   ```kotlin
   fun main() {
     val sum = { x: Int, y: Int -> x + y }
     println(sum(1, 2))
   }
   ```
  - 람다를 변수에 저장할 때는 타입을 추론할 문맥이 존재하지 않기 때문에 파라미터 타입 명시.

 ### 2. 람다식 직접 호출
   ```kotlin
    fun main() {
      {println(42)}()
    }
  ```
  - 읽기 어렵고 그다지 쓸모없음.
  - 람다 만들자마자 바로 호출(`{...}()`) < 람다 본문의 코드를 직접 실행(`run`)
    - run을 쓰는 호출에 아무 부가 비용이 들지 않으며 기본 구성요소와 비슷한 성능을 낸다는데 자세한건 10.2절에서 설명...
### 3. run 라이브러리 함수 이용
- 람다 본문의 코드를 실행
```kotlin
fun main() {
 run { println(42) }
}
```
- 식이 필요한 부분에서 코드 블록(몇가지 설정 or 다른 추가 작업)을 실행하고 싶을 때 유용

<br/>

- 코틀린에는 함수 호출 시 맨 뒤에 있는 인자가 람다식이라면 람다를 괄호 밖으로 빼낼 수 있음.
  - `people.maxByOrNull({p: Person -> p.age})` -> `people.maxByOrNull() { p: Person -> p.age }`
- 람다가 어떤 함수의 유일한 인자이고, 괄호 뒤에 람다를 썼다면 빈 괄호 없앨 수 있음.
  - `people.maxByOrNull() { p: Person -> p.age }` -> `people.maxByOrNull { p: Person -> p.age }`
- 마지막 인자만 람다인 경우 람다를 밖으로 빼내는 스타일 추천
- 람다가 둘 이상이라면 모두 괄호안에 넣는 것을 추천
```kotlin
  data class Person(val name: String, val age: Int)
  
   fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    val names = people.joinToString(
    	separator = " ",
    	transform = { it.name }
    )
    println(names)
    println(people.joinToString(" ") {it.name} )
  }
```
![image](https://github.com/user-attachments/assets/15c888bd-5732-4e1f-ac3a-8c521ec5c4e4)

- 컴파일러가 람다 파라미터의 타입도 추론가능하기 때문에 처음에는 타입을 쓰지 않고 람다를 작성하고 컴파일러가 타입을 모르겠다고 불평하는 경우에만 타입을 명시하자!

> it를 사용하는 관습은 코드를 아주 간단하게 만들어주지만 남용x,
> 람다 안에 람다가 내포되는 경우나 문맥에서 람다 파라미터의 의미나 타입을 쉽게 알 수 없는 경우에 명시적으로 선언하는 것이 도움이 됨.

- 람다 본문이 여러 줄로 이뤄진 경우 본문의 맨 마지막에 있는 식이 람다의 결괏값(return 필요X)

## 현재 영역에 있는 변수 접근(변수 캡쳐)
- 람다를 함수 안에서 정의하면 함수의 파라미터뿐 아니라 람다 정의보다 앞에 선언된 로컬 변수까지 람다에서 사용 가능.
  ```kotlin
  fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it")
    }
  }
  
  fun main() {
      val errors = listOf("403 Forbidden", "404 Not Found")
      printMessagesWithPrefix(errors, "Error:")
  }
  ```
![image](https://github.com/user-attachments/assets/ae639c23-6b85-467e-91b7-8ed48eed4c4d)
- 자바와 달리 코틀린 람다 안에서는 final이 아닌 변수에도 접근이 가능해서 바깥의 변수 변경가능하다.
  ```kotlin
  fun printProblemCounts(responses: Collection<String>) {
      var clientErrors = 0
      var serverErrors = 0
      responses.forEach {
          if (it.startsWith("4")) {
              clientErrors++
          } else if (it.startsWith("5")) {
              serverErrors++
          }
      }
      println("$clientErrors client errors, $serverErrors server errors")
  }
  
  fun main() {
      val responses = listOf("200 OK", "418 I'm a teapot", "500 Internal Server Error")
      printProblemCounts(responses)
  }
  ```
- **람다가 캡쳐한 변수** : 람다 안에서 접근할 수 있는 외부 변수
- 어떤 함수가 자신의 로컬 변수를 캡처한 람다를 반환하거나 다른 함수에 저장한다면 로컬 변수와 함수의 생명주기가 달라질 수 있음
  - 캡처한 변수가 있는 람다를 저장해서 함수가 끝난 뒤에 실행해도 람다의 본문 코드를 여전히 캡처한 변수를 읽거나 쓸 수 있음.
- 람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우 로컬 변수 변경은 람다가 실행될 때만 일어난다.
  ```kotlin
  fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.onClick {clicks++}
    return clicks
  }
  ```
  - 위 함수는 항상 0을 반환. 카운터 변수 밖으로 옮겨야 함.
  
## 멤버 참조(::)
- 이중콜론(::)을 사용하여 함수를 값으로 바꿀 수 있다.
- 멤버 참조는 정확히 한 메서드를 호출하거나 한 프로퍼티에 접근하는 함수 값을 만들어준다.
  ![image](https://github.com/user-attachments/assets/ca652e84-48a8-4a20-b910-2d6e194a7502)
- 람다가 인자가 여럿인 다른 함수에게 작업을 위임하는 경우 멤버 참조를 제공하면 편리(파라미터 이름과 함수를 반복하지 않아도 됨.)
### 생성자 참조
- ::뒤에 클래스 이름을 넣으면 생성자 참조를 만들 수 있다.
- 생성자 참조를 사용하면 클래스 생성 작업을 연기하거나 저장해둘 수 있다.
- 생성자를 함수처럼 다루기 위해 사용


## 값과 엮인 호출 가능 참조(bounded callable reference)
![image](https://github.com/user-attachments/assets/58bf76ea-0a64-49b0-a587-17f33e20ef87)

---
# 2. 자바의 함수형 인터페이스 사용 : 단일 추상 메서드
- 코틀린 프로젝트에 자바로 작성한 라이브러리를 사용하게 될 가능성이 상당히 큼.
- 코틀린 람다가 자바 API와 완전히 호환
- 단일추상메서드(SAM) 인터페이스인 경우 람다의 파라미터는 인터페이스의 유일한 추상 메서드의 파라미터와 대응.
- 자바 API에는 Runnable, Callable 등의 함수형 인터페이스와 이들을 활용하는 메서드가 많음.
- 코틀린은 함수형 인터페이스를 파라미터로 받는 자바 메서드를 호출할 때 람다를 사용할 수 있게 해주며, 이로 인해 코틀린 코드가 깔끔하고 전형적인 코틀린 스타일로 남을 수 있다.
## 람다를 자바 메서드의 파라미터로 전달
- 함수형 인터페이스를 파라미터로 받는 모든 자바 메서드에 람다 전달 가능
  - `void postponeComputation(int delay, Runnable computation);`이라는 Runnable을 인자로 받는 메서드가 있을 때, <br/>
    `postponeComputation(1000) { println(42) }`처럼 사용 가능.(컴파일러가 람다를 Runnable 인스턴스로 변환)
- 람다가 자신이 정의된 함수의 변수에 접근하지 않는다면 함수가 호출될 때마다 람다에 해당하는 익명객체가 재사용되지만, <br/>
  람다가 자신을 둘러싼 환경의 변수를 캡처하면 호출마다 새로운 인스턴스를 만들고, 그 객체 안에 캡처한 변수를 저장한다.
- 람다를 inline 표시가 돼있는 코틀린 함수에 전달하면 익명 클래스가 생성되지 않는다. + 대부분의 라이브러리 함수에는 inline이 붙어있다.

## SAM 변환 : 람다를 함수형 인터페이스로 명시적 변환
- SAM 생성자는 컴파일러가 생성한 함수로 람다를 단일추상메서드 인터페이스의 인스턴스로 명시적으로 변환해준다.
- 함수형 인터페이스의 인스턴스를 반환하는 함수가 있을 때, 람다를 직접 반환할 수 없으므로 SAM 생성자로 감싸줘야 함.
- 람다로 생성한 함수형 인터페이스 인스턴스를 변수에 저장해야 하는 경우에도 SAM 생성자 사용
  ```kotlin
  val listener = OnClickListener { view ->
    val text = when (view.id) {
      button1.id -> "First button"
      button2.id -> "Second button"
      else -> "Unknown button"
    }
    toast(text)
  }
  buttoñ1.setOnClickListener(listener)
  buttoñ2.setOnClickListener(listener)
  ```
> 람다에는 익명 객체와 달리 인스턴스 자신을 가리키는 this가 없다.
> 람다 안에서 this는 그 람다를 둘러싼 클래스의 인스턴스를 가리킨다.
> 
> 이벤트 리스너가 이벤트를 처리하다가 자기 자신의 리스너 등록을 해제해야 한다면 람다를 사용할 수 없다.
> 그런 경우 람다 대신 익명 객체를 사용해 리스너 구현. 익명 객체 안에서는 리스너를 해제하는 API 함수에 this를 넘길 수 있음.

---
# 3. 코틀린에서 SAM 인터페이스 정의: fun interface
- `fun interface`를 정의하여 스스로의 함수형 인터페이스 정의 가능.
- 코틀린의 함수형 인터페이스는 정확히 하나의 추상 메서드만 포함하지만 다른 비추상 메서드를 여럿 가질 수 있음.
  - 함수 타입의 시그니처에 들어맞지 않는 여러 복잡한 구조를 표현할 수 있음.
```kotlin
fun interface IntCondition {
  fun check(i: Int): Boolean // 추상 메서드는 정확히 하나만 존재
  fun checkString(s: String) = check(s.toInt())
  fun checkChar(c: Char) = chaeck(c.digitToInt())
}

fun main() {
  val isOdd = IntCondition { it % 2 != 0 }
  println(isOdd.check(1))
  println(isOdd.checkString("2"))
  println(isOdd.checkChar('3'))
}
```
- fun interface라고 정의된 타입의 파라미터를 받는 함수가 있을 때 람다 구현이나 람다에 대한 참조를 직접 넘길 수 있다.(동적으로 인터페이스 구현을 인스턴스화)
  ```kotlin
  fun checkCondition(i: Int, condition: IntCondition): Boolean {
    return condition.check(i)
  }
  fun main() {
    println(checkCondition(1) { it % 2 != 0 })
    
    val isOdd2: (Int) -> Boolean = { it % 2 != 0 }
    println(checkCondition(1, isOdd2))
  }
  ```

---
# 4. 수신 객체 지정 람다: with, apply, also
- 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메서드를 호출할 수 있게 하는 것.
## with 함수
### with를 사용한 리팩토링
```kotlin
fun alphabet(): String {
  val result = StringBuilder()
  for (letter in 'A'..'Z') {
      result.append(letter)
  }
  result.append("\nNow I Know the alphabet!")
  return result.toString()
}

fun main() {
    println(alphabet())
}
```
- result라는 이름을 반복사용 -> with
```kotlin
fun alphabet(): String {
  val stringBuilder = StringBuilder()
  return with(stringBuilder) {
      for (letter in 'A'..'Z') {
          append(letter)
      }
      append("\nNow I Know the alphabet!")
      toString()
  }
}

fun main() {
    println(alphabet())
}
```
- **with**
  - 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다.
  - 람다 내에서는 this참조를 통해 수신 객체에 접근 가능(this 생략 가능)
```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
      append(letter)
    }
    append("\nNow I Know the alphabet!")
    toString()
}

fun main() {
    println(alphabet())
}
```
> ### 메서드 이름 충돌
> with에 인자로 넘긴 객체의 클래스와 with코드가 들어있는 클래스 안에 이름이 같은 메서드가 있는 경우
> this참조 앞에 레이블을 붙이면 호출하고 싶은 메서드를 명확하게 정할 수 있다.
> ex) `this@OuterClass.toString()`

- with가 반환하는 값은 람다 코드를 실행한 결과, 람다의 결과 대신 수신 객체가 필요한 경우 apply 라이브러리 함수 사용

## apply 함수
- `with`와 거의 동일하게 작동.
- 항상 자신에 전달된 객체(수신 객체) 반환
```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString()

fun main() {
    println(alphabet())
}
```
- apply를 임의의 타입의 확장 함수로 호출 가능.
- apply를 호출한 객체는 람다의 수신 객체가 된다.
- 객체 초기화에 유용
  ```kotlin
  fun createViewWithCustomAttributes(context: Context) =
    TextView(context).apply {
      text = "Sample Text"
      textSize = 20.0
      setPadding(10, 0, 0, 0)
    }
  ```
### 표준 라이브러리 함수
- buildString(표준 라이브러리)
  - 위의 alphabet코드에서 StringBuilder객체를 만드는 일과 toString() 호출하는 일을 알아서 해줌.
  ```kotlin
  fun alphabet() = buildString {
    for (letter in 'A'..'Z') {
      append(letter)
    }
    append("\nNow I know the alphabet!")
  }
  ```
- 컬렉션 빌더 함수(buildList, buildSet, buildMap) : 읽기 전용 컬렉션을 생성하지만 생성과정에서는 가변 컬렉션인 것처럼 다룰 수 있음.

## 객체에 추가 작업 수행: also
- apply와 마찬가지로 수신 객체를 받으며, 그 수신 객체에 대해 어떤 동작을 수행한 후 수신 객체를 돌려줌.
- also의 람다 안에서는 수신 객체를 인자로 참조(파라미터 이름을 부여 or *it*사용
- 객체 자체의 프로퍼티나 함수를 다루는 동작이 아니라 **원래의 수신 객체를 인자로 받는 동작을 실행**할 때 유용
### also를 사용해 효과를 추가로 적용하기
```kotlin
fun main() {
    val fruits = listOf("Apple", "Banana", "Cherry")
    val uppercaseFruits = mutableListOf<String>()
    val reversedLongFruits = fruits
    	.map { it.uppercase() }
        .also { uppercaseFruits.addAll(it) }
        .filter {it.length > 5 }
        .also { println(it) }
        .reversed()
}
```

---
# 요약
- 람다를 사용하면 코드 조각을 다른 함수에게 인자로 넘길 수 있고, 공통 코드 구조를 쉽게 추출할 수 있다.
- 코틀린에서는 람다가 함수 인자인 경우 괄호 밖으로 람다를 빼내서 코드를 더 깔끔하고 간결하게 만들 수 있다.
- 람다의 인자가 단 하나뿐인 경우 인자 이름을 지정하지 않고 it라는 디폴트이름으로 부를 수 있다.
- 람다는 외부 변수를 캡처할 수 있다.
- 메서드, 생성자, 프로퍼티의 이름 앞에 ::을 붙이면 각각에 대한 참조를 만들 수 있다. 그런 참조를 람다 대신 다른 함수에 넘길 수 있다.
- filter, map, all, any 등의 함수를 활용하면 직접 원소를 이터레이션하지 않고 컬렉션에 대한 대부분의 연산을 수행할 수 있다.
- 추상 메서드가 단 하나뿐인 인터페이스(SAM 인터페이스)를 구현하기 위해 그 인터페이스를 구현하는 객체를 생성하는 대신 람다를 직접 넘길 수 있다.
- 수신 객체 지정 람다를 사용하면 람다 안에서 미리 정해둔 수신 객체의 메서드를 직접 호출할 수 있다. 이 람다의 본문은 그 본문을 둘러싼 코드와는 다른 콘텍스트에서 작동하기 때문에
  코드를 구조화할 때 도움이 된다.
- with함수를 사용하면 어떤 객체에 대한 참조를 반복해서 언급하지 않으면서 그 객체의 메서드를 호출할 수 있다.
- apply를 사용하면 어떤 객체이든 빌더 스타일의 API를 사용해 생성, 초기화할 수 있다.
- also를 사용하면 객체에 대해 추가 작업을 수행할 수 있다.





Q1. 
```kotlin
class Button {
    private var onClickListener: (() -> Unit)? = null

    fun onClick(listener: () -> Unit) {
        onClickListener = listener
    }

    fun simulateClick() {
        onClickListener?.invoke()
    }
}

fun setupCounter(button: Button): Int {
    var count = 0
    button.onClick {
        count++
        println("Clicked! Count: $count")
    }
    return count
}

fun main() {
    val button = Button()
    val clickCount = setupCounter(button)

    println("Initial clickCount: $clickCount") // 초기값 출력

    button.simulateClick()
    button.simulateClick()

    println("Final clickCount: $clickCount") // 최종값 출력
}
```
위 코드의 실행 결과는?

Q2.
```kotlin
import java.util.concurrent.Callable

// Runnable 을 받는 foo
fun foo(action: Runnable) {
    println("▶ Runnable foo:")
    action.run()
}

// Callable<Int> 을 받는 foo
fun foo(action: Callable<Int>) {
    println("▶ Callable<Int> foo, returned: ${action.call()}")
}

fun main() {
    // 아래 한 줄을 주석 해제하면 컴파일 에러(모호성) 발생:
    foo { println("hi") }
}
```
위 코드의 실행 결과는?
.
.
.
.
.
.
.
.
.
.
.
.
.
A2.
컴파일 오류
```kotlin
// SAM 생성자를 명시적으로 써서 모호성 해소
foo(Runnable { println("Hello from Runnable!") })
foo(Callable<Int> {
    println("Hello from Callable!")
    999
})
```

