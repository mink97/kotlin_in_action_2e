<aside>
💡

기본 아이디어: 널이 될 수 있는 타입을 컴파일 타임에 미리 감지한다.

</aside>

# 널이 될 수 있는 타입으로 널이 될 수 있는 변수 명시

## **기본적으로 null 금지**

- `String`은 기본적으로 `null`을 허용하지 않음.
- `null`을 넘기면 컴파일 에러 발생.
    
    ```kotlin
    fun strLen(s: String) = s.length
    strLen(null) // ❌ 컴파일 에러
    ```
    

## **널 허용 타입은 `?`를 붙임**

- `String?`, `Int?` 등으로 표시.
- null이 될 수 있는 값을 허용.
    
    ```kotlin
    fun strLenSafe(s: String?) = if (s != null) s.length else 0
    ```
    

## **널 허용 타입의 제약**

- `s.length` 같은 직접 접근 불가능 → 컴파일 에러
- null 검사 후에만 안전하게 접근 가능

## **null 검사를 통한 타입 추론**

- `if (s != null)` 같은 검사 후에는 `s`를 non-null로 간주함

## **널이 될 수 있는 값은 널이 아닌 곳에 대입 불가**

```kotlin
val x: String? = null
val y: String = x // ❌ 타입 불일치 에러
```

# **타입이란 무엇인가?**

> 타입 = 값의 집합 + 허용되는 연산의 집합
> 
- **David Parnas의 정의 (1976)**: 타입은 "가능한 값들과 해당 값들에 적용할 수 있는 연산들"의 집합
- 예시:
    - `Double`: 모든 부동소수점 수 + 수학 연산 가능
    - `String`: 문자열 값 + 문자열 메서드 호출 가능

## **자바의 문제점 – null 처리의 허술함**

- `String` 타입 변수에 **null**이 들어갈 수 있음 → **값에 따라 호출 가능한 연산이 달라짐**
- 타입이 정해졌음에도 불구하고 null 체크 없이 메서드 호출하면 `NullPointerException` 발생

## 자바의 해결책들 (불완전함)

- `@Nullable`, `@NotNull` 어노테이션 + 정적 분석 도구
    - 표준 아님 → 적용 일관성 부족
- `Optional` (Java 8 도입)
    - null을 명시적으로 감쌈 → **불편하고 성능 저하**, **외부 라이브러리와 호환 문제**

## **코틀린의 해결책 – 타입 수준에서 null 명시**

- `String?` → null을 허용하는 명시적 타입
- null 가능 여부에 따라 **가능한 연산이 컴파일 시점에 결정**
    - null일 가능성이 있는 값에는 안전한 방식 (`?.`, `?:`, `!!`)으로만 접근 가능
    - **컴파일 단계에서 오류를 방지** → 런타임 오류 예방

## **런타임 성능은 동일**

- `String?`은 내부적으로 래퍼 타입(`Optional`)이 아님
- null 검사 외에 별도 **성능 오버헤드 없음**
    - 코틀린은 null 처리를 **언어 차원**에서 하되, **실행 시점 성능 손해는 없다**

# 안전한 호출 연산자 `?.`

## 개념

- **`?.`는 null 체크 + 호출을 동시에 수행하는 연산자**
- `str?.uppercase()`는 다음과 같음:
    
    ```kotlin
    if (str != null) str.uppercase() else null
    ```
    

## 동작 방식

- `?.` 앞의 값이 **null이 아니면** → 일반 메서드 호출
- `?.` 앞의 값이 **null이면** → **호출은 무시되고 null 반환**
- 반환 타입도 **널이 될 수 있는 타입**

```kotlin
fun printAllCaps(str: String?) {
    val allCaps: String? = str?.uppercase()
    println(allCaps)
}
```

```kotlin
printAllCaps("abc")  // 출력: ABC
printAllCaps(null)   // 출력: null
```

## 프로퍼티에도 사용 가능

```kotlin
class Employee(val name: String, val manager: Employee?)

fun managerName(employee: Employee): String? =
    employee.manager?.name
```

## 연쇄 호출 가능

```kotlin
class Address(val street: String, val zip: Int, val city: String, val country: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
    val country = this.company?.address?.country
    return if (country != null) country else "Unknown"
}
```

- `person?.company?.address?.country`처럼 중첩 구조에서도 안전하게 접근 가능
- **null이 중간에 하나라도 있으면 전체 표현식 결과는 null**

# 엘비스 연산자 `?:`란?

> null인 경우 기본값을 지정할 수 있는 연산자
> 

```kotlin
val result = a ?: b
```

- `a`가 **null이 아니면** → `a` 반환
- `a`가 **null이면** → `b` 반환

> 이름 유래: ?: 모양을 시계방향 90도 돌리면 Elvis Presley 얼굴처럼 보인다고 해서 붙여짐
> 

## 기본값 지정 예시

```kotlin
fun greet(name: String?) {
    val recipient: String = name ?: "unnamed"
    println("Hello, $recipient!")
}
```

- `name`이 null → `"unnamed"` 출력됨

## 안전한 호출과 함께 사용

```kotlin
fun strLenSafe(s: String?): Int = s?.length ?: 0
```

- `s`가 null이면 `length`를 호출하지 않고 `0` 반환

## 표현 간소화: 중첩된 null 처리 → 한 줄로

```kotlin
fun Person.countryName() = company?.address?.country ?: "Unknown"
```

## `throw`나 `return`과 함께 사용 가능

```kotlin
val address = person.company?.address
    ?: throw IllegalArgumentException("No address")
```

- `company`나 `address`가 없으면 즉시 예외 발생

## 전체 예시 요약

```kotlin
fun printShippingLabel(person: Person) {
    val address = person.company?.address
        ?: throw IllegalArgumentException("No address")

    with(address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}
```

- null이면 NPE 대신 의미 있는 예외 발생
- `with`를 사용해 중복된 변수 접근 줄임

# `as?` — 안전한 타입 캐스트 연산자

## 개념

- `as?`는 **지정한 타입으로 캐스트**를 시도함
- **성공** → 변환된 값 반환
- **실패** → `null` 반환 (예외 발생 ❌)

```kotlin
val p = obj as? Person
```

## 자바/코틀린 기본 캐스트 (`as`)와 비교

| 연산자 | 실패 시 동작 | 예시 |
| --- | --- | --- |
| `as` | 예외 발생 (`ClassCastException`) | `val p = obj as Person` |
| `as?` | `null` 반환 | `val p = obj as? Person` |

## 실용 예제: `equals` 구현

```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false
        return otherPerson.firstName == firstName &&
               otherPerson.lastName == lastName
    }

    override fun hashCode(): Int =
        firstName.hashCode() * 37 + lastName.hashCode()
}
```

- `as?`로 타입 확인 + 캐스트 → 실패 시 `return false`
- `as?` 뒤에 `?: return` 패턴은 코틀린에서 자주 사용됨

## 장점

- **예외 없는 안전한 타입 체크**
- **한 줄로 타입 검사 + 캐스트 + 기본값 처리 가능**
- 스마트 캐스트와 함께 사용 시 **간결하고 안전한 코드** 작성 가능

# 널 아님 단언(Not-null assertion)

## 개념

- `!!`는 **널이 될 수 있는 값**을 강제로 **널이 아닌 값으로 간주**하게 만듦
- **실제로 null이면 `NullPointerException` 발생**

```kotlin
val strNotNull: String = str!!
```

> !! = 컴파일러에게 “이 값은 null이 아님을 내가 보장할게!”라고 강하게 주장하는 도구
> 

## 예제

```kotlin
fun ignoreNulls(str: String?) {
    val strNotNull: String = str!!
    println(strNotNull.length)
}

ignoreNulls(null) // ❌ KotlinNullPointerException 발생
```

## 🔥 주의사항

- **안전하지 않은 연산**이므로 **남용 금지**
- 실제 null이면 `!!`가 있는 **위치에서 예외 발생**
- **스택 트레이스에는 줄 번호만 표시**되고, 어떤 식에서 null이었는지 구분 어려움
- ❌ **이런 형태는 절대 지양**
    
    ```kotlin
    person.company!!.address!!.country
    ```
    

## 실전 사용 예시: 조건 검사는 외부에서 완료되었지만 컴파일러는 모를 때

```kotlin
class CopyRowAction(val list: SelectableTextList) {
    fun isActionEnabled(): Boolean =
        list.selectedIndex != null

    fun executeCopyRow() {
        val index = list.selectedIndex!! // 외부에서 검사했다고 확신할 때
        val value = list.contents[index]
        // 클립보드에 복사
    }
}
```

- `!!` 없이 안전하게 하려면:
    
    ```kotlin
    val index = list.selectedIndex ?: return
    val value = list.contents[index]
    ```
    

# `let` 함수란?

> null이 아닐 때만 실행되는 람다 블록을 정의할 수 있는 스코프 함수
> 
- 주로 `?.let { ... }` 형태로 사용
- null이 아닌 경우에만 블록 실행
- 블록 안에서는 수신 객체를 `it` 또는 별도 이름으로 사용 가능

## 기본 구조

```kotlin
val email: String? = "example@domain.com"
email?.let { sendEmailTo(it) }  // email이 null이 아닐 때만 호출됨
```

## 장점

- **null 체크 + 값 전달 + 실행**을 한 줄로 처리 가능
- 값이 null이 아닌 경우만 **함수 호출이나 로직 실행** 가능
- 별도의 변수 없이 **식 체이닝**을 간결하게 처리 가능

## 예제

### 1. 널 안전한 함수 호출

```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

val email: String? = "yole@example.com"
email?.let { sendEmailTo(it) }  // 출력: Sending email to yole@example.com
```

### 2. 긴 식 결과 처리

```kotlin
getTheBestPersonInTheWorld()?.let { sendEmailTo(it.email) }
```

### 3. null인 경우는 아무 동작도 하지 않음

```kotlin
val email: String? = null
email?.let { sendEmailTo(it) }  // 아무 일도 일어나지 않음
```

## 기타 스코프 함수 비교 요약

| 함수 | 수신객체 접근 | 반환값 | 주용도 |
| --- | --- | --- | --- |
| `let` | `it` | 람다 결과 | null이 아닐 때 실행, 결과 전달 |
| `apply` | `this` | 수신 객체 | 설정(빌더 패턴 등) |
| `also` | `it` | 수신 객체 | 로그/추가 작업 후 원본 유지 |
| `run` | `this` | 람다 결과 | 초기화 + 결과 계산 |
| `with(x)` | `this` | 람다 결과 | 여러 호출 묶기 (non-null 객체) |

# 지연 초기화 프로퍼티

## 문제 상황: 생성자 밖에서 초기화해야 하는 경우

- 코틀린에서는 일반적으로 **모든 프로퍼티는 생성자에서 초기화**해야 함
- 하지만 다음과 같은 상황에서는 **나중에 값이 설정됨**
    - Android `onCreate()`
    - JUnit `@BeforeAll`, `@BeforeEach`
    - DI 프레임워크 (예: Dagger, Guice)

```kotlin
private var myService: MyService? = null  // 널 허용 타입 사용 시
```

- 이런 경우 사용 시마다 `!!` 또는 `?.`를 사용해야 해서 코드가 지저분함

## 해결책: `lateinit var`

- **널이 아닌 타입이지만 나중에 초기화**할 수 있도록 허용
- `!!` 없이 바로 사용 가능 (단, 초기화 전 접근 시 예외 발생)

```kotlin
private lateinit var myService: MyService

@BeforeAll
fun setUp() {
    myService = MyService()
}

@Test
fun testAction() {
    assertEquals("Action Done!", myService.performAction())
}
```

## `lateinit` 특징 요약

| 항목 | 설명 |
| --- | --- |
| 대상 | **`var`만 가능** (`val`은 안 됨) |
| 타입 | **널이 될 수 없는 타입**만 사용 가능 |
| 초기화 전 접근 시 | `UninitializedPropertyAccessException` 발생 |
| 어디에 사용 가능? | 클래스 멤버, 최상위 프로퍼티, 지역 변수 |

## 장점

- 널이 될 수 있는 타입(`?`)을 피하면서도 유연한 초기화 가능
- DI 프레임워크와의 호환성 탁월
- **예외 메시지가 명확**함 (`lateinit property X has not been initialized`)

# 널이 될 수 있는 타입에 대한 확장

## 기본 개념

```kotlin
fun String?.isNullOrBlank(): Boolean =
    this == null || this.isBlank()
```

- `String?` 타입에 대한 확장
- `this`는 null일 수 있음 → **직접 null 체크 필요**
- null인 경우 `true` 반환, 아니면 `isBlank()` 호출

## 사용 예시

```kotlin
fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) {
        println("Please fill in the required fields")
    }
}
```

- `input`이 null이든, 공백이든 → 조건 충족
- `?.` 없이 `input.isNullOrBlank()` 형태로 호출 가능!

## 내부 동작 이해

```kotlin
fun String?.isNullOrBlank(): Boolean =
    this == null || this.isBlank()
```

- `this == null` → true
- 아니라면 `this.isBlank()` 호출
- **스마트 캐스트**로 `this`가 null 아님을 자동 인식

## let과의 차이점

```kotlin
val recipient: String? = null
recipient.let { sendEmailTo(it) } // ❌ it은 String? → 컴파일 에러

recipient?.let { sendEmailTo(it) } // ✅ 안전한 호출 필요
```

- `let`은 호출 전에 수신 객체가 null인지 검사하지 않음
- 따라서 **`?.let { ... }` 형태로 사용**해야 안전

## 언제 써야 하나?

- 수신 객체가 null일 수도 있는 상황에서 호출 가능한 메서드를 제공하고 싶을 때
- **null-safe 유틸리티 함수** 제공: `isNullOrEmpty()`, `isNullOrBlank()`, `orEmpty()` 등

# 타입 파라미터는 **기본적으로 null 허용**

> T라는 타입 파라미터가 있다고 해서 null이 될 수 없는 것은 아님!
> 
- `fun <T> doSomething(t: T)`의 `T`는 사실상 `T: Any?`와 같음
- 즉, **기본적으로 `T`는 nullable**

## 예제 1: null을 허용하는 타입 파라미터

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode())
}

printHashCode(null)  // 출력: null
```

- 여기서 `T`는 `Any?`로 추론됨
- `t?.hashCode()`는 **안전한 호출 연산자** 덕분에 null일 경우도 처리됨

## 🚫 null을 받지 않게 하려면?

> T: Any처럼 상한(bounded type)을 명시해야 함
> 

## 예제 2: null을 거부하는 타입 파라미터

```kotlin
fun <T : Any> printHashCode(t: T) {
    println(t.hashCode())  // ? 없이 호출 가능
}

printHashCode(42)     // ✅ 출력됨
printHashCode(null)   // ❌ 컴파일 오류
```

- `T: Any` → `T`는 **non-null 타입**만 가능
- null을 넘기면 **컴파일 오류 발생**

# 널 가능성과 자바

## 자바와 코틀린의 널 시스템 차이

- **코틀린**: 널 가능성은 **타입 시스템에 통합**됨 → 컴파일러가 null 관련 오류 감지
- **자바**: 널 가능성은 **문서나 어노테이션 수준에 한정**됨 → 컴파일 시점에는 불명확

## 자바 → 코틀린: 널 가능성 해석

| 자바 어노테이션 | 코틀린 타입으로 변환 |
| --- | --- |
| `@Nullable` | `Type?` |
| `@NotNull` | `Type` |
| 없음 | **플랫폼 타입 (`Type!`)** |

> ❗ 어노테이션이 없으면 플랫폼 타입으로 간주됨 → 널 여부 알 수 없음
> 

## 플랫폼 타입 (Platform Type)

- 자바에서 가져온 타입 중 널 여부 정보가 **명확하지 않은 타입**
- **코틀린에서는 `Type!`로 표기됨 (IDE에서만)**
- 다음 두 가지 **모두 허용**:
    
    ```kotlin
    val s1: String = person.name      // null 아님으로 간주
    val s2: String? = person.name     // null 허용으로 간주
    ```
    

### 특징

- null 검사 없이도 접근 가능하지만 **NPE 위험 있음**
- null 검사 또는 `?:`, `?.` 같은 연산자를 **개발자가 직접 사용해야 함**

## 예제

```kotlin
fun yellAt(person: Person) {
    println(person.name.uppercase() + "!!!")
}

yellAt(Person(null))
// ❌ NPE 발생
```

```kotlin
fun yellAtSafe(person: Person) {
    println((person.name ?: "Anyone").uppercase() + "!!!")
}
// ✅ 안전하게 처리
```

## 자바 타입 오버라이드 시 주의

```java
// 자바 인터페이스
interface StringProcessor {
    void process(String value);
}
```

```kotlin
// 코틀린 구현 – 두 가지 다 가능
class StrictPrinter : StringProcessor {
    override fun process(value: String) { println(value) } // null 불가
}

class SafePrinter : StringProcessor {
    override fun process(value: String?) {
        if (value != null) println(value)
    }
}
```

- 자바 코드가 null을 넘기면 `StrictPrinter`는 런타임 예외 발생
- **null 전달 가능성을 고려해 `String?`로 선언**하면 더 안전함

# 코틀린 널 처리 요약 정리

- **기본 타입은 null을 허용하지 않음** → `String`은 null 불가, `String?`은 가능
- **널 처리 도구들**:
    - `?.` : **안전한 호출** – null이면 전체 식 결과가 null
    - `?:` : **엘비스 연산자** – null이면 기본값/예외/return 처리
    - `!!` : **널 아님 단언** – null 아님을 개발자가 보장, 실패 시 NPE
    - `let` : **스코프 함수** – null이 아닐 때만 블록 실행 (`?.let { ... }`)
    - `as?` : **안전한 캐스트** – 실패 시 null 반환, 예외 없음
- **확장 함수**로 null-safe한 연산 정의 가능 (`String?.isNullOrBlank()` 등)
- **타입 파라미터 `T`는 기본적으로 nullable**
- **자바 타입은 플랫폼 타입 (`Type!`)** → null 여부를 알 수 없음, 직접 책임져야 함

# 오늘의 퀴즈

> 아래 코드를 실행하면 어떤 결과가 출력될까?
> 
> 
> 이유도 설명해보세요.
> 

```kotlin
fun describe(input: String?) = input?.uppercase() ?: "UNKNOWN"

fun main() {
    println(describe("hello"))
    println(describe(null))
}
```

## 정답:

```
HELLO
UNKNOWN
```

## 해설:

- `"hello"`는 null 아님 → `uppercase()` 호출 → `"HELLO"`
- `null`은 `?.`로 인해 `null` 반환 → `?:`로 `"UNKNOWN"` 대체됨
