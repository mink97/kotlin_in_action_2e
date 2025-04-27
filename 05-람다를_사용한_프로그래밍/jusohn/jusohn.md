# 05장 - (p.237 - p.)

## 람다식과 맴버 참조

람다: 코드 블록을 값처럼 다루기

- 필요할 때 바로 만들어 쓸 수 있는 함수
- 코드 블록을 변수에 저장하거나 넘길 수 있어 추상화와 반복되는 코드를 크게 줄일 수 있음
- 코틀린은 람다를 1등 시민 (First-class citizen) 으로 취급함

```kotlin
button.setOnClickListener(object: OnClickListener) {
  override fun onClick(v: View) {
    println("I was clicked!")
  }
}

// 람다로 구현
button.setOnClickListener {
  println("I was clicked!")
}
```

- 위와 같이 람다로 자바 익명 내부 클래스와 같은 동작을 하지만 훨씬 더 간단한 코드를 작성 가능

```kotlin
data class Person(val name: String, val age: Int)

// 가장 나이가 많은 사람은?
fun main() {
  val people = listOf(Person("Alice", 29), Person("Bob", 31))
  println(people.maxByOrNull { it.age })
  // Person(name=Bob, age=31)

  // 혹은
	people.maxByOrNull(Person::age)
}
```

- 람다가 파라메터 (인자) 가 1개일 경우, `it` 이라는 암시적 이름을 사용함

![Screenshot 2025-04-27 at 8.21.25 PM.png](attachment:b6e33d62-083b-4940-8e44-bdea526adbe1:Screenshot_2025-04-27_at_8.21.25_PM.png)

- 람다식을 직접 호출할 수도 있음
  - 굳이? 본문의 코드를 직접 실행하는 편이 나음.
  - 코드의 일부분을 블록으로 감싸 실행할 필요가 있다면, `run` 을 사용
    - `run` 은 인자로 받은 람다를 실행해주는 라이브러리 함수

```kotlin
// 원본
people.maxByOrNull({ p: Person -> p.age })

// 코틀린에는 함수 호출 시 맨 뒤의 인자가 람다식이라면, 그 람다를 괄호 밖으로 빼낼 수 있음
people.maxByOrNull() { p: Person -> p.age }

// 컴파일러는 람다 파라미터의 타입도 추론할 수 있음
// 따라서 파라미터 타입을 명시할 필요가 없음
people.maxByOrNull() { p -> p.age }

// 람다가 어떤 함수의 유일한 인자이고, 괄호 뒤에 람다를 썼다면 호출 시 빈 괄호를 없에도 됨
// 유일한 파라미터 이름을 암시적 it 으로 대체 가능
people.maxByOrNull { it.age }

// 맴버 참조를 통해 더 짧게 사용할 수도 있음
people.maxByOrNull(Person::age)
```

- 람다 안에 람다가 내포되는 경우, 각 람다의 파라미터를 명시하는 편이 좋음.
  - 명시하지 않을 경우 각각의 it 이 가르키는 파라미터가 어떤 람다에 속하는지 파악하기 어려울 수 있음.

```kotlin
val getAge = { p: Person -> p.age }
people.maxByOrNull(getAge)
```

- 람다를 변수에 저장할 때는 파라미터의 타입을 추론할 문맥이 존재하지 않으므로, 파라미터 타입을 명시해야 함.

## 변수 캡쳐

```kotlin
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
  messages.forEach {
		println("$prefix $it")
  }
}

fun main() {
  val errors = listOf("403 Forbidden", "404 Not Found")
  printMessagesWithPrefix(errors, "Error:")
  // Error: 403 Forbidden
  // Error: 404 Not Found
}
```

![image.png](attachment:1a272c3e-0c91-42a9-acf8-8bbe35ffabfa:image.png)

- 람다를 함수 안에서 정의하면, 함수의 파라미터뿐 아니라 람다 정의보다 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있음

- 코틀린과 자바 람다의 다른 점
  - 코틀린 람다 안에서는 파이널 변수가 아닌 변수에 접근할 수 있음
  - 따라서 람다 안에서 바깥의 변수를 변경해도 됨

```kotlin
fun printProblemCounts(responses: Collection<String>) {
  var clientErrors = 0
  var severErrors = 0
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
  // 1 client errors, 1 serer errors
}
```

- 위 예제에서, `prefix`, `clientErrors`, `serverErrors` 와 같이 람다 안에서 접근할 수 있는 외부 변수를 ‘람다가 캡처한 변수’ 라고 부름

- 기본적으로 함수 안에 정의된 로컬 변수의 생명주기는 함수가 반환되면 끝남
  - 하지만 어떤 함수가 자신의 로컬 변수를 캡처한 람다를 반환하거나 다른 변수에 저장한다면, 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있음
  - 변수가 있는 람다를 저장해서 함수가 끝난 뒤에 실행해도 람다의 본문 코드는 여전히 캡처한 변수를 읽거나 쓸 수 있음
- 어떻게?

  - 파이널 변수를 캡처한 경우
    - 람다 코드를 변수 값과 함께 저장
  - 파이널이 아닌 변수를 캡처한 경우
    - 변수를 특별한 래퍼로 감싸서 나중에 변경하거나 읽을 수 있게 한 다음, 래퍼에 대한 참조를 람다 코드와 함께 저장

- 람다를 이벤트 핸들러나 다른 비동기적으로 실행되는 코드로 활용하는 경우, 로컬 변수 변경은 람다가 실행될 때만 일어남

```kotlin
fun tryToCountButtonClicks(button: Button): Int {
  var clicks = 0
  button.onClick { clicks++ }
  return clicks
}
```

- 위 코드는 버튼 클릭 횟수를 제대로 샐 수 없음 (항상 0을 반환)
- onClick 핸들러는 호출될 때마다 clicks 의 값을 증가시키지만, 그 값의 변경을 관찰할 수는 없음
  - 핸들러는 tryToCountButtonClicks 가 clicks 를 반환한 다음에 호출되기 때문
  - 위 함수를 제대로 구현하려면, 클릭 횟수를 세는 카운터 변수를 함수 바깥으로 옮겨야 함

## 맴버 참조

- 함수를 값으로 바꿀 때 이중 콜론(::) 을 사용
- :: 을 사용하는 식은 맴버 참조라고 불림

```kotlin
fun salute() = println("Salute!")

fun main {
  run(::salute)
  // Salute!
}
```

## 생성자 참조

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
  val createPerson = ::Person
  val p createPerson("Alice", 29)
  println(p)
  // Person(name=Alice, age=29)
}
```

- 클래스 생성 작업을 연기하거나 저장해 둘 수 있음
- 확장 함수도 맴버 함수와 똑같은 방식으로 참조 가능

```kotlin
fun Person.isAdult() = age => 21
val perdicate = Person::isAdult
```

```kotlin
fun main() {
  val seb = Person("Sebastian", 26)
  val personsAgeFunction = Person::age
  println(personsAgeFunction(seb))
  // 26
  val sebsAgeFunction = seb::age
  println(sebsAgeFunction())
  // 26
}
```

## SAM 인터페이스 정의: `fun interface`

## 수신 객체 지정 람다: with, apply, also

- with
  - 대상 객체 스코프를 만듬
- apply
  - 초기화에 사용
- also
  - 중간 사이드 이펙트 발생에 사용

## 문제 1

- 아래 코드는 왜 실행되지 않을까요?

  ```kotlin
  fun salute() = println("Salute!")

  fun main() {
    run(salute)
  }
  ```

## 문제 2

- 아래 코드를 보고, makeRunningAverage 를 구현하세요.

  ```kotlin
  fun main() {
    val avg = makeRunningAverage()

    println(avg(10.0))   // 10.0   (첫 번째 값)
    println(avg(20.0))   // 15.0   ((10 + 20) / 2)
    println(avg(0.0))    // 10.0   ((10 + 20 + 0) / 3)
  }
  ```
