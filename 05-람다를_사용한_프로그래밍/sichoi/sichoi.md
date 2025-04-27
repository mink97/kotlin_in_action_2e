람다(lambda): (aws labmda 아님) 다른 함수에 넘길 수 있는 작은 코드 조각

- **멤버 참조와 람다의 관계**
- 람다와 함께 자바 API와 라이브러리 사용
- 함수형 인터페이스 사용 방법
- 수신 객체 지정 람다

# 람다식과 멤버 참조

## “코드 블록을 값으로 다룬다.”

- 일련의 동작을 변수에 저장하거나 다른 함수에 넘겨야 하는 경우
    - 익명 내부 클래스 목적을 달성
        - but. 번거로움
    - 함수를 값처럼 다루기!
        - 인스턴스를 함수에 넘기는 것이 아닌 함수 자체를 다른 함수에 전달

<aside>
💡

**코틀린의 함수형 프로그래밍 특징 요약**

- **일급 시민인 함수**:
    
    → 함수를 변수에 저장하거나, 파라미터로 전달하거나, 함수가 다른 함수를 반환할 수 있다. (람다 지원)
    
- **불변성 (Immutability)**:
    
    → 객체가 만들어진 이후에는 내부 상태가 변하지 않도록 설계한다.
    
- **부수 효과 없음 (No Side Effects)**:
    
    → 함수는 같은 입력에 대해 항상 같은 출력을 내며, 외부 상태를 변경하지 않는다. (순수 함수, pure function)
    
</aside>

### [기존 방식: 객체로 구현하기]

**코틀린 (object 선언 사용)**

```kotlin
downloadFile(url, object : DownloadListener {
    override fun onSuccess(file: File) {
        println("Download completed: ${file.name}")
    }

    override fun onFailure(error: String) {
        println("Download failed: $error")
    }
})

```

- `DownloadListener`는 `onSuccess`랑 `onFailure` 두 개 메서드를 가진 인터페이스
- 파일이 잘 내려받아졌으면 `onSuccess`, 실패하면 `onFailure`가 호출되는 구조
- 매번 `object : DownloadListener {}`를 쓰는 게 귀찮고 코드가 길어지는 문제

### [람다로 개선한 버전: 단일 메서드일 때!]

**만약 인터페이스에 메서드가 하나만 있다면? → 람다로 훨씬 간결하게 가능**

```kotlin
fun interface SimpleDownloadListener {
    fun onComplete(file: File)
}
```

**람다 사용**

```kotlin
downloadFile(url) { file ->
    println("Downloaded file: ${file.name}")
}
```

- `object :` 구문 없이 그냥 **중괄호 `{}` 안에 동작**만 넣으면 끝
- 파라미터 이름(`file`)도 알아서 타입 추론

| 전통적 방식 (object) | 람다 방식 |
| --- | --- |
| 길고 중첩된 코드 | 짧고 직관적 |
| 인터페이스 이름을 매번 명시 | 그냥 동작만 정의 |
| 메서드 여러 개일 때 필요 | 메서드 하나면 람다로 대체 가능 |

## 람다와 컬렉션

### [기존 방식: 직접 for 루프 돌리기]

```kotlin
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

```

- 직접 루프 돌면서 최대 나이 찾아야 해서 **코드가 길고 실수하기 쉬움**.

### [개선된 방식: `maxByOrNull` 사용]

```kotlin
fun main() {
    val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.maxByOrNull { it.age })
}

```

- **`maxByOrNull { it.age }`** 사용하면,
    
    → "나이(age)가 가장 큰 사람"을 자동으로 찾아줌.
    
- `it`은 컬렉션 원소를 가리키는 **암시적 변수 이름**.

## 람다식 문법

### 1. 기본 문법

- **람다식**은 항상 `{}` 중괄호로 감싼다.
- **형태**: `{ 파라미터 -> 본문 }`
- **예시**:
    
    ```kotlin
    val sum = { x: Int, y: Int -> x + y }
    println(sum(1, 2))  // 3
    ```
    

### 2. 람다를 바로 호출하기

- 만들자마자 호출 가능: `{ println(42) }()`
- 하지만 보통은 `run { ... }`을 사용해서 더 깔끔하게 호출한다.
    
    ```kotlin
    run { println(42) }
    ```
    

### 3. 함수에 람다 전달하기

- 람다는 **함수의 인자**로 넘길 수 있다.
- 마지막 인자가 람다면, 괄호 바깥으로 뺄 수 있다.
    
    ```kotlin
    people.maxByOrNull { it.age }
    ```
    
- 이건 다음과 다 같은 의미다:
    
    ```kotlin
    people.maxByOrNull({ p: Person -> p.age })
    people.maxByOrNull() { p: Person -> p.age }
    ```
    

### 4. 타입 생략

- 컴파일러가 타입을 추론할 수 있으면 **파라미터 타입 생략 가능**.
    
    ```kotlin
    people.maxByOrNull { p -> p.age }
    ```
    
- 파라미터가 하나뿐이면 **it**을 쓸 수 있다.
    
    ```kotlin
    people.maxByOrNull { it.age }
    ```
    

### 5. 람다를 변수에 저장할 때

- **변수에 저장하는 람다**는 **파라미터 타입을 꼭 명시**해야 한다.
    
    ```kotlin
    val getAge = { p: Person -> p.age }
    people.maxByOrNull(getAge)
    ```
    

### 6. 여러 줄로 구성된 람다

- 람다 본문이 여러 줄일 수 있다.
- 마지막 줄의 값이 **람다의 반환값**이 된다.
    
    ```kotlin
    val sum = { x: Int, y: Int ->
        println("Computing the sum...")
        x + y
    }
    println(sum(1, 2))  // 3
    ```
    

## 현재 영역의 변수 접근

### 1. 코틀린 람다는 바깥 변수를 자유롭게 사용 가능

- 함수 파라미터, 로컬 변수 모두 **읽고**, 심지어 **수정**도 할 수 있다.

```kotlin
fun printMessagesWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it") // prefix는 바깥 함수 파라미터
    }
}
```

- `prefix`는 바깥 함수의 **파라미터**인데, 람다 안에서 자유롭게 사용한다.

### 2. 코틀린은 **var(변경 가능한 변수)** 도 캡처 가능

- 자바는 파이널(final) 또는 사실상 파이널(변경되지 않는) 변수만 람다에서 쓸 수 있다.
- **코틀린은 var도 가능!** → 람다 안에서 값을 바꿀 수 있다.

```kotlin

fun countErrors(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0

    responses.forEach {
        if (it.startsWith("4")) clientErrors++
        else if (it.startsWith("5")) serverErrors++
    }

    println("$clientErrors client errors, $serverErrors server errors")
}

```

- `clientErrors`, `serverErrors`는 `var`로 선언됐고, **람다 안에서 직접 수정**하고 있다.
- 문제 없이 작동함!

**자바에서는 이렇게 못함**

```java
void countErrors(List<String> responses) {
    int clientErrors = 0; // Error: 람다 안에서 수정 불가

    responses.forEach(response -> {
        if (response.startsWith("4")) clientErrors++; // 컴파일 에러
    });
}
```

- 자바에서는 **clientErrors가 final 또는 effectively final**이어야 해서 수정 불가.
- 해결하려면 `AtomicInteger` 같은 별도 객체를 써야 한다.

### 3. 내부 동작 (코틀린)

- `val`을 캡처하면 값 복사
- `var`를 캡처하면 내부적으로 **Ref**(래퍼 객체)를 만들어서 값을 바꿀 수 있게 한다

```kotlin
class Ref<T>(var value: T)

fun main() {
    val counter = Ref(0)
    val inc = { counter.value++ }  // 내부적으로 이런 식으로 래핑
}
```

- 실제 코틀린 코드에서는 Ref를 직접 만들 필요는 없고, 알아서 해준다.

### 4. 주의할 점: **비동기 실행**에서는 변수 변경 즉시 관찰 불가

- 함수가 끝난 후에 람다가 실행될 경우, 로컬 변수 변경은 의미가 없다.

**문제 예시**

```kotlin
fun tryToCountButtonClicks(button: Button): Int {
    var clicks = 0
    button.setOnClickListener { clicks++ }
    return clicks // 항상 0을 반환함
}
```

- 클릭 이벤트는 함수 종료 후에 발생해서 `clicks`는 변한 걸 볼 수 없음.
- 해결: clicks를 **클래스 프로퍼티**로 옮기기.

```kotlin
class ClickCounter {
    var clicks = 0

    fun bind(button: Button) {
        button.setOnClickListener { clicks++ }
    }
}

fun main() {
	val counter = ClickCounter()
	counter.bind(button)
	
	// 나중에 클릭 수 확인
	println(counter.clicks)
}
```

## 람다에서 클래스 멤버 참조

### 1. 멤버 참조란?

- **이미 있는 함수나 프로퍼티를** 람다처럼 넘기고 싶을 때 사용하는 문법.
- **`::` (이중 콜론)** 을 사용한다.

### 2. 기본 사용법

- 클래스 이름 + `::` + 멤버 이름

```kotlin
val getAge = Person::age  // 프로퍼티 참조
```

```kotlin
val getAge = { person: Person -> person.age } // 이와 동등
```

```kotlin
people.maxByOrNull(Person::age) // 이렇게도 사용가능
```

### 3. 최상위 함수 참조

- 최상위에 선언된 함수도 `::함수이름` 으로 참조할 수 있다.

```kotlin
fun salute() = println("Salute!")

fun main() {
    run(::salute)
}
```

- `run`은 받은 함수를 실행한다.

### 4. 생성자 참조

- 클래스를 `::클래스이름` 으로 참조하면 **생성자 호출**을 저장할 수 있다.

```kotlin
val createPerson = ::Person
val p = createPerson("Alice", 29)
println(p)  // Person(name=Alice, age=29)
```

### 5. 확장 함수도 참조 가능

- 확장 함수도 일반 멤버처럼 참조할 수 있다.

```kotlin
fun Person.isAdult() = age >= 21

val predicate = Person::isAdult
```

- `predicate`는 `Person` 인스턴스를 받아 `isAdult()` 호출하는 함수처럼 작동.

## 값과 엮인 호출 가능 참조

### 1. 기존 멤버 참조 vs 값과 엮인 참조

| 구분 | 특징 |
| --- | --- |
| `클래스::멤버` | → **인스턴스를 인자로** 받아 멤버를 참조 |
| `인스턴스::멤버` | → **인스턴스가 이미 고정**되어 있어서 **인자 없이** 호출 가능 |

### 2. 예시 코드

```kotlin
fun main() {
    val seb = Person("Sebastian", 26)

    // 일반 멤버 참조: Person 객체를 받아서 age를 반환
    val personsAgeFunction = Person::age
    println(personsAgeFunction(seb))  // 26

    // 값과 엮인 호출 가능 참조: seb 인스턴스에 묶여 있음
    val sebsAgeFunction = seb::age
    println(sebsAgeFunction())  // 26
}
```

### 3. 간단 비교

| 코드 | 설명 |
| --- | --- |
| `Person::age` | Person을 넘겨야 age를 얻을 수 있음 |
| `seb::age` | seb에 이미 묶여 있으므로 그냥 호출 `()` |

→ `seb::age`는 사실상 `{ seb.age }` 람다를 더 간결하게 표현한 것과 같음.

# 자바 함수형 인터페이스 사용: 단일 추상 메소드

- 코틀린 프로젝트에서 자바로 작성한 라이브러리를 사용할 가능성이 큼.
- 코틀린은 이런 부분이 잘 호환되게 만들어짐.
- 이런 호환에 대한 예시를 설명하는 챕터.

- **자바 인터페이스에 메서드가 하나만 있으면** (`Single Abstract Method`, **SAM** 인터페이스라고 부름)
    - AWS의 SAM(Serverless Application Model이 아니다.)
- **코틀린에서는 람다**를 바로 넘길 수 있다.

```java
public interface OnClickListener {
    void onClick(View v);
}
```

```kotlin
button.setOnClickListener { view ->
    println("I was clicked!")
}
```

- `OnClickListener`의 `onClick(View v)` 메서드가
    
    → 코틀린 람다의 `view` 파라미터와 매칭됨.
    

## 람다를 자바 메소드 파라미터로 전달

### 1. 자바 함수형 인터페이스에 코틀린 람다 전달 가능

- 자바 메서드가 **함수형 인터페이스**(메서드 하나짜리 인터페이스)를 파라미터로 받으면,
- 코틀린에서는 그냥 **람다**를 넘기면 된다.

- **Runnable 이란?**
    
    **Runnable**은 **"할 일을 캡슐화한 인터페이스”**
    
    (쉽게 말해 "할 일 하나를 객체처럼 만들어서 나중에 실행할 수 있게" 해주는 것)
    
    ```java
    public interface Runnable {
        void run();  // 추상 메서드 하나만 있음
    }
    ```
    
    - `run()`이라는 메서드 하나만 있다.
    - **"이 안에 할 일을 써라"** 라는 뜻.
    
    **Runnable을 왜 쓰는 걸까?**
    
    - 어떤 작업을 **나중에 실행**하고 싶을 때.
    - 특히 **스레드(thread)** 같은 걸 사용할 때 많이 씀.
    
    (하지만 꼭 스레드에만 쓰는 건 아냐. 그냥 "나중에 실행"하고 싶은 거 캡슐화할 때 다 씀!)
    
    **Runnable 없이 직접 실행할 때**
    
    ```kotlin
    println("Doing something!")
    ```
    
    - 그냥 바로 실행돼버림.
    
    **Runnable로 감싸서 실행할 때**
    
    ```kotlin
    val r = Runnable { println("Doing something!") } // 할 일(println)을 묶어놓음
    r.run()  // 나중에 필요할 때 run() 호출해서 실행
    ```
    
    - "할 일"을 `Runnable`로 저장해뒀다가, `run()`을 호출할 때 실행

**자바 메서드**

```java
void postponeComputation(int delay, Runnable computation);
```

**코틀린 호출**

```kotlin
postponeComputation(1000) { println(42) }
```

- 컴파일러가 자동으로 **람다를 Runnable 인스턴스**로 변환해준다.

### 2. 명시적 익명 객체 방식과 차이

- 직접 **익명 객체**를 만들 수도 있다.
    
    ```kotlin
    postponeComputation(1000, object : Runnable {
        override fun run() {
            println(42)
        }
    })
    ```
    
- 하지만 **람다**를 쓰면 훨씬 간결하고,
    
    캡처하지 않는 경우 **람다 인스턴스를 재사용**해서 성능도 좋다.
    

### 3. 캡처 여부에 따른 인스턴스 관리

| 경우 | 결과 |
| --- | --- |
| 람다가 **외부 변수 캡처 안함** | 같은 Runnable 인스턴스 재사용 |
| 람다가 **외부 변수 캡처함** | 호출마다 새로운 인스턴스 생성 |

### 예시 (캡처하는 경우)

```kotlin
fun handleComputation(id: String) {
    postponeComputation(1000) {
        println(id)  // id를 캡처 → 매번 새 Runnable 인스턴스 생성
    }
}
```

## 람두를 함수형 인터페이스로 명시적 변환 (SAM 변환)

### 1. SAM 변환이란?

- **람다를 명시적으로 함수형 인터페이스 인스턴스로 감싸는 것**.
- 사용할 때는 **SAM 생성자**(`인터페이스이름(람다)`)를 쓴다.

### 2. SAM 변환이 필요한 경우

- **람다를 반환하거나 변수에 저장**해야 할 때!
- 컴파일러가 자동 변환해줄 수 없는 문맥에서는 **직접 변환**이 필요.

**함수에서 람다를 반환할 때**

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}

fun main() {
    createAllDoneRunnable().run()  // 출력: All done!
}
```

- `Runnable { ... }` → `Runnable` 인터페이스 인스턴스로 변환.

**람다를 변수에 저장할 때**

```kotlin
val listener = OnClickListener { view ->
    val text = when (view.id) {
        button1.id -> "First button"
        button2.id -> "Second button"
        else -> "Unknown button"
    }
    toast(text)
}

button1.setOnClickListener(listener)
button2.setOnClickListener(listener)

```

- `listener` 하나를 만들어 여러 버튼에 재사용!

### 4. 주의할 점

- **람다에는 this가 없음**:
    - 람다 안에서 `this`는 **람다를 둘러싼 클래스**를 가리킨다.
    - 만약 **리스너 해제** 같은 작업(자기 자신을 참조해야 하는 경우)이 필요하면, **람다 대신 익명 객체**를 써야 한다.
- **오버로드 함수가 모호할 때**도 SAM 생성자를 명시적으로 써야 한다.

# 코틀린에서 SAM 인터페이스 정의: fun interface

## 1. fun interface란?

- 코틀린에서도 **직접 함수형 인터페이스**를 만들 수 있게 해주는 키워드.
- **"추상 메서드가 딱 하나만"** 있어야 한다.

## 2. 기본 예시

```kotlin
fun interface IntCondition {
    fun check(i: Int): Boolean  // 추상 메서드 (하나만 있어야 함)

    fun checkString(s: String) = check(s.toInt())  // 비추상 메서드는 추가 가능
}
```

- `check`는 반드시 구현해야 하고,
- `checkString`, `checkChar` 같은 **비추상 메서드**는 추가할 수 있다.

## 3. 사용 예시

```kotlin
val isOdd = IntCondition { it % 2 != 0 }  // 람다로 간단히 구현
println(isOdd.check(1))        // true
println(isOdd.checkString("2")) // false
println(isOdd.checkChar('3'))   // true
```

- 람다 하나로 `IntCondition` 인터페이스를 구현할 수 있다.

## 4. 함수를 호출할 때 fun interface 파라미터로 넘기기

```kotlin
fun checkCondition(i: Int, condition: IntCondition): Boolean {
    return condition.check(i)
}

checkCondition(1) { it % 2 != 0 }  // 직접 람다로 전달
```

## 5. fun interface의 장점 (특히 자바와 호환할 때)

- **fun interface**를 쓰면 코틀린과 자바 간 호출 코드가 **더 깔끔**해진다.
- 코틀린 함수 타입(`(String) -> Unit`)을 자바에서 쓰면 `Unit.INSTANCE`를 명시적으로 리턴해야 하지만, **fun interface**를 쓰면 그런 불편 없이 간단하게 람다만 넘기면 된다.

### 비교 예시 (자바 코드)

```java
// fun interface 사용
MainKt.consumeHello(s -> System.out.println(s.toUpperCase()));

// 함수 타입 사용
MainKt.consumeHelloFunctional(s -> {
    System.out.println(s.toUpperCase());
    return Unit.INSTANCE;  // 꼭 명시해야 함 (귀찮음)
});

```

# `⭐️ 수신 객체 지정 람다: with, apply, also ⭐️`

- 이번 챕터는 코틀린에서 정말 편리하고 자주 사용되는 람다 함수를 다룬다.
- 동작 원리에 대한 이해가 정말 중요하다고 느껴져서 별표시로 강조했다.

## `with` 함수 정리

### 1. `with`란?

- **수신 객체(this)를 지정하는 람다**를 실행하게 해주는 표준 라이브러리 함수.
- **문법상 언어 키워드처럼 보이지만**, 실제로는 단순한 **일반 함수**다.
- 객체 이름을 **반복하지 않고** 여러 메서드를 연속 호출할 수 있게 한다.

### 2. `with` 기본 구조

```kotlin
with(수신객체) {
    // this를 통해 (혹은 생략하고) 수신 객체의 메서드, 프로퍼티에 접근
}
```

- 첫 번째 인자: 수신 객체
- 두 번째 인자: 수신 객체를 **this**로 사용하는 람다 (IntelliJ에서 this를 클릭하면 중괄호를 가리키게 된다.)

### 3. 예시 비교

**일반 방식 (수신 객체 이름 반복)**

```kotlin
val sb = StringBuilder()
for (letter in 'A'..'Z') {
    sb.append(letter)
}
sb.append("\nNow I know the alphabet!")
println(sb.toString())
```

**with 사용 (이름 생략 가능)**

```kotlin
val sb = StringBuilder()
return with(sb) {
    for (letter in 'A'..'Z') {
        append(letter)  // this.append()에서 this 생략 가능
    }
    append("\nNow I know the alphabet!")
    toString()  // 마지막 식이 with 전체 결과로 반환
}
```

- 람다 안에서 **this**는 `sb`를 가리킨다.
- **this.** 생략해도 된다.

### 4. 작동 방식 요약

| 항목 | 설명 |
| --- | --- |
| 첫 번째 인자 | 수신 객체 (ex. StringBuilder) |
| 두 번째 인자 | 그 객체를 대상으로 실행할 람다 |
| 반환값 | **람다 본문 마지막 식**의 결과가 반환된다 |
- `with(stringBuilder)` 안에서 `toString()` 호출 → 그 결과가 `with` 전체 결과가 됨.

### 5. `this` 충돌 문제

- 만약 `with` 안과 바깥 클래스 둘 다에 같은 이름의 메서드가 있다면,
    
    **`this@레이블` 문법**을 써서 어떤 `this`를 쓸지 명확히 해야 한다.
    

```kotlin
class OuterClass {
    override fun toString() = "OuterClass"

    fun alphabet() = with(StringBuilder()) {
        // StringBuilder의 toString()을 호출
        toString()

        // 바깥 클래스(OuterClass)의 toString()을 호출하고 싶으면
        this@OuterClass.toString()
    }
}
```

- `this@OuterClass.toString()` → 바깥쪽 클래스의 메서드 호출.

### 6. 간결한 형태로 리팩터링

- **식 본문 함수**(`=`)를 써서 더 짧게 쓸 수 있다.

```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    toString()
}
```

- `StringBuilder()`를 즉시 with의 인자로 넘기고,
- `return` 키워드 없이 마지막 식(`toString()`)이 결과가 된다.

### 7. 주의할 점

- `with`는 **람다 블록의 마지막 식 값**을 반환한다.
    
    → 수신 객체 자체를 반환하는 게 아님.
    
- 만약 **수신 객체 그 자체**를 반환하고 싶다면 `apply`를 써야 한다 (5.4.2에서 배움).

## `apply` 함수 정리

### 1. `apply`란?

- **`with`와 비슷하게** 수신 객체(this)를 지정해 여러 작업을 수행하는 함수.
- 하지만 **`apply`는 항상 호출한 수신 객체 자체를 반환**한다.

2. `apply` 기본 구조

```kotlin
수신객체.apply {
    // this를 통해 객체 조작
}

```

- 람다 블록 안에서 수신 객체(this)를 사용.
- **apply 함수의 결과 = 수신 객체 그 자체**

### 3. `with` vs `apply` 차이

| 항목 | with | apply |
| --- | --- | --- |
| **수신 객체 접근 방법** | this (생략 가능) | this (생략 가능) |
| **반환값** | 람다 블록의 마지막 식 결과 | **수신 객체 자신** |
| **호출 방식** | `with(객체) { ... }` | `객체.apply { ... }` |
| **주 사용 목적** | 여러 작업 후 최종 결과 반환 | 객체를 구성(초기화) 후 반환 |

**apply 사용해서 알파벳 만들기**

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}.toString()
```

- `StringBuilder()`를 만들고
- `apply` 블록 안에서 알파벳을 쌓은 후
- `apply` 결과로 다시 `StringBuilder`를 받아서 `toString()` 호출

### 4. 실제 활용: 객체 생성 + 초기화

**TextView 예시 (안드로이드)**

```kotlin
fun createViewWithCustomAttributes(context: Context) = TextView(context).apply {
    text = "Sample Text"
    textSize = 20.0f
    setPadding(10, 0, 0, 0)
}
```

- TextView를 만들자마자
- apply 블록 안에서 여러 프로퍼티를 세팅
- **세팅이 끝난 TextView 인스턴스**가 반환됨

### 5. 확장: build 계열 함수들

- **`buildString`**, **`buildList`**, **`buildSet`**, **`buildMap`** 처럼
- 코틀린 표준 라이브러리는 `apply` 패턴을 활용한 **우아한 빌더 함수**들도 제공한다.

**buildString**

```kotlin
fun alphabet() = buildString {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
}
```

- `StringBuilder`를 알아서 만들어주고
- 끝나면 `.toString()`까지 자동으로 해줌

**buildList**

```kotlin
val fibonacci = buildList {
    addAll(listOf(1, 1, 2))
    add(3)
    add(index = 0, element = 3)
}
```

**buildSet**

```kotlin
val fruits = buildSet {
    add("Apple")
    if (true) {
        addAll(listOf("Banana", "Cherry"))
    }
}
```

**buildMap**

```kotlin
val medals = buildMap<String, Int> {
    put("Gold", 1)
    putAll(listOf("Silver" to 2, "Bronze" to 3))
}
```

- 모두 가변 객체처럼 작업한 뒤, 최종 결과는 **불변 객체**로 얻는다.

## ✅ `also` 함수 정리

### 1. `also`란?

- **apply와 비슷하게** 객체에 대해 어떤 동작을 수행한 다음,
- **수신 객체 자신을 그대로 반환**하는 함수.
- **다른 점**: also는 람다 안에서 객체를 **it(기본 이름)** 으로 참조한다.

### 2. 기본 구조

```kotlin
수신객체.also { it ->
    // it을 통해 수신 객체를 다룬다
}
```

- 여기서 `it`은 수신 객체를 가리킨다.
- **객체의 속성을 수정**하기보다는,
    
    **"부가적인 일(로깅, 저장 등)"을 수행**할 때 쓰는 것이 자연스럽다.
    

### 3. `apply` vs `also` 비교

| 항목 | apply | also |
| --- | --- | --- |
| **수신 객체 접근** | this로 접근 (생략 가능) | it으로 접근 (또는 직접 이름 붙이기) |
| **주로 하는 일** | 객체를 구성(초기화) | 부수 작업 추가(로그 찍기, 저장 등) |
| **반환 값** | 수신 객체 | 수신 객체 |

**기본 예시**

```kotlin
val x: List<Int> = listOf(1, 2, 3).also {
    println("List is: $it")
}
```

- 리스트를 그대로 반환하면서, **중간에 출력하는 부가 작업**을 추가했다.

**과일 컬렉션 변환 예시**

```kotlin
fun main() {
    val fruits = listOf("Apple", "Banana", "Cherry")
    val uppercaseFruits = mutableListOf<String>()

    val reversedLongFruits = fruits
        .map { it.uppercase() }                // 대문자 변환
        .also { uppercaseFruits.addAll(it) }    // 변환된 결과를 다른 리스트에 저장
        .filter { it.length > 5 }               // 5글자 넘는 것만 필터링
        .also { println(it) }                   // 필터링 결과 출력
        .reversed()                             // 리스트 뒤집기

    println(uppercaseFruits)    // [APPLE, BANANA, CHERRY]
    println(reversedLongFruits) // [CHERRY, BANANA]
}
```

- **`.also {}`** 안에서 부수 작업(리스트에 추가하거나 출력)을 하고,
- 체이닝(메서드 연결)을 계속 이어간다.

### 5. 정리 포인트

- `also`는 **"이 객체를 유지하면서 중간에 무언가 추가로 하고 싶을 때"** 사용.
- apply처럼 this가 아니라 **it으로 참조**.
- 사용 예시: **로깅, 값 저장, 디버깅, 부가 처리 추가**.

| 항목 | with | apply | also |
| --- | --- | --- | --- |
| **수신 객체 접근** | this (생략 가능) | this (생략 가능) | it (또는 직접 이름 지정) |
| **반환 값** | 람다 블록의 마지막 식 결과 | 수신 객체 자신 | 수신 객체 자신 |
| **주로 하는 일** | 여러 작업을 하고 최종 결과 반환 | 객체를 구성(초기화) | 부수 작업 추가 (로깅, 저장 등) |
| **호출 방식** | `with(객체) { ... }` | `객체.apply { ... }` | `객체.also { ... }` |

# 챕터 요약

## 📚 5장 요약: 람다를 사용한 프로그래밍

### 1. 람다의 기본

- **람다**는 **코드 조각을 값처럼 다룰 수 있는 기능**이다.
- 다른 함수에 **인자로 넘기거나**, **변수에 저장**할 수 있다.
- **함수 인자가 람다**이면, 괄호 밖으로 람다를 빼내서 코드를 더 깔끔하게 쓸 수 있다.

---

### 2. 람다 파라미터 다루기

- **람다의 인자가 하나**일 경우, 이름 없이 **it**으로 바로 접근할 수 있다.
- 긴 이름을 생략하고 간단한 람다를 작성할 수 있다.

---

### 3. 람다와 외부 변수 캡처

- 람다 안에서 **외부 변수**(바깥 함수의 지역 변수 등)를 **읽고 수정**할 수 있다.
- 이런 동작을 **변수 캡처**(capturing)라고 부른다.

---

### 4. 멤버 참조(::)

- **함수**, **생성자**, **프로퍼티**를 람다 대신 참조할 수 있다.
- 문법: `::함수이름`, `클래스이름::프로퍼티이름`
- 예시: `Person::age`, `::salute`

---

### 5. 컬렉션 연산에서 람다 사용

- `filter`, `map`, `all`, `any` 같은 고차 함수에 람다를 넘겨 **컬렉션을 간결하게 처리**할 수 있다.
- 직접 for 루프를 돌릴 필요 없이 선언적으로 작성할 수 있다.

---

### 6. 자바 SAM 인터페이스와 호환

- **SAM(Single Abstract Method)** 인터페이스(메서드 하나만 가진 인터페이스)는 코틀린에서 **람다로 대체 가능**하다.
- 코틀린은 자동으로 람다를 해당 인터페이스 구현체로 변환해준다.

---

### 7. 수신 객체 지정 람다

- **수신 객체**를 지정하면, 람다 안에서 해당 객체의 메서드나 프로퍼티를 **this**로 접근할 수 있다.
- 코드 구조를 깔끔하게 만들 수 있다.

---

### 8. with, apply, also 사용법

| 함수 | 설명 | 특징 |
| --- | --- | --- |
| `with(객체) {}` | 객체에 여러 작업을 수행 후 **결과** 반환 | 결과값이 필요할 때 사용 |
| `객체.apply {}` | 객체를 설정하고 **객체 자체** 반환 | 객체 초기화에 사용 |
| `객체.also {}` | 객체에 부수 작업 추가 후 **객체 자체** 반환 | 로깅, 디버깅 등 부가 작업에 사용 |

---

# ✨ 챕터 한 줄 요약

> 코틀린 람다는 코드를 값처럼 다루고, 외부 변수 캡처, 컬렉션 처리, 수신 객체 지정, 자바 함수형 인터페이스 호환까지 지원하여 간결하고 강력한 함수형 프로그래밍을 가능하게 한다!
> 

# 🧐 오늘의 깜짝 퀴즈

### ❓ 퀴즈 1

아래 코드는 컴파일이 될까? 실행하면 어떤 결과가 나올까?

```kotlin
fun main() {
    val printMessage = { message: String -> println("Message: $message") }

    val result = listOf(1, 2, 3)
        .map { it * 2 }
        .with("Hello") {
            printMessage(this)
        }
        .filter { it > 2 }

    println(result)
}
```

**선택지**

A) "Message: Hello" 출력 후, [2, 4, 6] 출력

B) "Message: Hello" 출력 후, [4, 6] 출력

C) 컴파일 에러

D) 런타임 에러 발생


### ❓ 퀴즈 2

다음 코드 실행 결과는?

```kotlin
fun main() {
    var count = 0
    val inc = { count++ }

    repeat(3, inc)
    println(count)
}
```

**선택지**

A) 0

B) 1

C) 3

D) 컴파일 에러


### ❓ 퀴즈 3

아래 코드의 출력 결과는?

```kotlin
fun main() {
    val fruits = listOf("apple", "banana", "cherry")
    val result = fruits
        .map { it.uppercase() }
        .also { it.filter { it.startsWith("A") } }

    println(result)
}
```

**선택지**

A) [APPLE]

B) [APPLE, BANANA, CHERRY]

C) []

D) 컴파일 에러
