# 7. nullable type

## 7.1 NulllPointerException(NPE)을 피하고 값이 없는 경우 처리: nallable
- 널 가능성은 NPE를 피할 수 있게 돕는 코틀린 타입 시스템의 특성
- null관련 문제를 실행시점에서 컴파일 시점으로 옮기려고 함.
- nullable여부를 타입 시스템에 추가, 여러가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성 줄일 수 있음.

## 7.2 nullable type으로 널이 될 수 있는 변수 명시
- "이 함수가 null을 인자로 받을 수 있는가?"
- 어떤 타입이든 타입 이름 뒤에 물음표(?)를 붙이면 그 타입의 변수나 프로퍼티에 null 참조를 저장할 수 있다.
- nullable type은 연산의 종류 제한
  - 널이 될 수 있는 타입 값의 메서드를 직접 호출 불가
  - 널이 될 수 있는 값을 널이 될 수 없는 타입의 변수에 대입 불가
  - 널이 될 수 있는 타입의 값을 널이 아닌 타입의 파라미터를 받는 함수에 전달 불가
- 일단 null과 비교하고 나면 컴파일러는 그 사실을 기억하고 null이 아님이 확실한 영역에서는 해당 값을 null이 아닌 타입의 값처럼 사용 가능

## 7.3 타입의 의미 자세히 살펴보기
- nullable타입과 non-null타입을 구분하면 각 타입의 값에 대해 어떤 연산이 가능할지 명확히 이해할 수 있고,
  실행 시점에 예외를 발생시킬 수 있는 연산을 판단할 수 있다.
- 그런 연산을 아예 금지시킬 수 있다.

## 7.4 안전한 호출 연산자로 null 검사와 메소드 합치기(안전한 호출): `?.`
- `?.`: null 검사와 메서드 호출을 한 연산으로 수행
- 호출하려는 값이 null이 아니라면 `?.`는 일반 메서드 호출처럼 작동.
- null이면 호출은 무시되고 결괏값은 null
![image](https://github.com/user-attachments/assets/3a1f1ffd-f9be-4558-9685-a190a80c78d7)

## 7.5 엘비스 연산자로 null에 대한 기본값 제공: `?:`
- **엘비스 연산자**: null 대신 사용할 기본값을 지정할 때 편리하게 사용할 수 있는 연산자
![image](https://github.com/user-attachments/assets/63c3b989-39cc-4102-b96f-866893271cb0)
- 코틀린에서는 return이나 throw등도 식이기 때문에 엘비스 연산자의 오른쪽에 return, throw등도 넣을 수 있어 더욱 편하게 사용가능
  - ?: 왼쪽 값이 null이 되면 함수가 즉시 return이나 throw

## 7.6 예외를 발생시키지 않고 안전하게 타입 캐스팅(안전한 캐스트 연산자): `as?`
- `as?`: 어떤 값을 지정한 타입으로 변환. 값을 대상 타입으로 변환할 수 없으면 null 반환
![image](https://github.com/user-attachments/assets/1a58e4bc-161b-478c-94c4-f633bbde77c9)
- 캐스트를 수행한 뒤에 엘비스연산자를 사용하는 것이 일반적인 패턴
```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false

        return otherPerson.firstName == firstName && otherPerson.lastName == lastName
    }

    override fun hashCode(): Int =
        firstName.hashCode() * 37 + lastName.hashCode()
}

fun main() {
    val person1 = Person("John", "Doe")
    val person2 = Person("John", "Doe")

    println(person1 == person2)
    println(person1 == null)
    println(person1.equals(null))
    println(person1.equals(42))
}
```
- 안전한 호출(`?.`), 안전한 캐스트(`as?`), 엘비스 연산자(`?:`)는 유용하기 때문에 코틀린 코드에 자주 나타남.

## 7.7 널 아님 단언: `!!`
- 어떤 값이든 널이 아닌 타입으로 바꿀 수 있음.
- 실제 null에 대해 !!적용하면 NPE 발생
- 호출된 함수가 언제나 다른 함수에서 null이 아닌 값을 전달받는다는 사실이 분명할 때(null 검사 다시 수행x)
- 여러 !!단언문을 한 줄에 함께 쓰는 일은 피하는 것이 좋다.(어떤 식에서 예외가 발생했는지 모름)

## 7.8 let 함수
- let 함수를 ?.연산자와 함꼐 사용하면 원하는 식을 평가해서 결과가 null인지 검사한 다음에 그 결과를 변수에 넣는 작업을 간단한 식을 사용해 한꺼번에 처리할 수 있다.
- 널이 될 수 있는 값을 널이 아닌 값만 인자로 받는 함수에 넘기는 경우에 자주 사용
- let 함수는 자신의 수식 객체를 인자로 전달받은 람다에 넘김.(it)
![image](https://github.com/user-attachments/assets/2e05ffe9-15a0-4652-b250-1ba83c59a1cd)
```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main() {
    var email: String? = "yole@example.com"
    email?.let {sendEmailTo(it)} // Sending email to yole@example.com
    email = null
    email?.let {sendEmailTo(it)}
}
```
- 긴 식의 값이 null이 아닐 때 수행해야 하는 로직이 있을 때 let을 쓰면 더 편함
  - ex) `getTheBestPersonInTheWorld()?.let {sendEmailTo(it.email)}
- 여러 값이 null인지 검사해야 할 때, let을 내포시켜 처리하면 코드가 복잡해져서 알아보기 어려워짐.(if를 사용해 모든 값을 한꺼번에 처리하는 편이 나음)
> [!NOTE]
> with, apply, let, run, also
> |함수|x를 어떤 방식으로 참조하는가|반환값|
> |---|---|---|
> |x.let { ... } |it|람다 결과|
> |x.also { ... } |it|x|
> |x.apply { ... }|this|x|
> |x.run { ... }|this|람다 결과|
> |with(x) { ... } |this|람다 결과|
> - 다루는 객체가 null이 아닌 경우에만 코드 블록을 실행하고 싶다. -> let + ?.
> - 어떤 식의 결과를 변수에 담되 그 영역을 한정시키고 싶다. -> let
> - 빌더 스타일의 API(ex. 인스턴스 생성)를 사용해 객체 프로퍼티를 설정할 때 -> apply
> - 객체에 어떤 동작을 실행한 후 원래의 객체를 다른 연산에 사용하고 싶을 때 -> also
> - 하나의 객체에 대해 이름을 반복하지 않으면서 여러 함수 호출을 그룹으로 묶고 싶을 때 -> with
> - 객체를 설정한 다음에 별도의 결과를 돌려주고 싶을 때 -> run

## 7.9 직접 초기화하지 않는 널이 아닌 타입: 지연 초기화 프로퍼티(lateinit)
- 객체를 일단 생성 후에 나중에 전용메서드를 통해 초기화하는 프레임워크 많음.
  - android: onCreate / JUnit: @BeforeAll, @BeforeEach
- 코틀린에서 클래스 안의 널이 아닌 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메서드 안에서 초기화할 수 없음.
- 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 함.
- `lateinit`변경자를 붙이면 프로퍼티 나중에 초기화가능(생성자 안에서 초기화할 필요 없음.)
- lateinit 프로퍼티는 항상 var이여야 함.
![image](https://github.com/user-attachments/assets/371132f6-e916-4514-9b82-f4f9dfc6b5cd)
- 다양한 자바 프레임워크와의 호환성을 위해 lateinit이 지정된 프로퍼티와 가시성이 똑같은 필드 생성
  - lateinit 프로퍼티가 public이라면 코틀린이 생성한 필드도 public

## 7.10 안전한 호출 연산자 없이 타입 확장: nullable type에 대한 확장
- 메서드 호출이 null을 수신 객체로 받고 내부에서 null 처리 가능(단, 확장함수에서만)
- 일반 멤버 호출은 해당 인스턴스가 null인지 여부 검사하지 않음.
- 안전한 호출(?.)없이도 nullable 수신 객체 타입에 대해 선언된 확장 함수 호출 가능. null 값이 들어오는 경우 적절히 처리
- 널이 될 수 있는 타입(`type?`)에 대한 확장을 정의하면 널이 될 수 있는 값에 대해 그 확장 함수를 호출할 수 있다.
- 내부에서 this는 null이 될 수 있음. 명시적으로 null 여부 검사 필요
- 널이 될 수 있는 타입의 확장함수 안에서는 this가 null이 될 수 있다는 점이 자바와 다름.
- let은 this가 null인지 검사 안함. nullable값에 대해 안전한 호출을 사용하지 않고 let을 호출하면 람다의 인자는 nullable type으로 추론
- 직접 확장함수를 정의한다면 그 확장함수를 nullable 타입에 대해 정의할지 여부를 고민할 필요 있음.
  - 처음에는 널이 될 수 없는 타입에 대한 확장 함수를 정의
 
## 7.11 타입 파라미터의 널 가능성
- 타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 nullable type
- 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상계(upper bound) 지정
  ```kotlin
  fun<T: Any> printHashCode(t: T) {
    println(t.hashCode())
  }
  ```
- 타입 파라미터(T)는 널이 될 수 있는 타입을 표시하려면 반드시 물음표를 타입 이름 뒤에 붙여야 한다는 규칙의 유일한 예외

## 7.12 널 가능성과 자바
- 자바 타입 시스템은 널 가능성 지원하지 않음.
- 널 가능성 어노테이션이 없는 경우 자바의 타입은 코틀린의 **플랫폼 타입**이 됨.
### 플랫폼 타입
- 코틀린이 널 관련 정보를 알 수 없는 타입
- 플랫폼 타입에 대해 수행하는 모든 연산에 대한 책임이 온전히 개발자에게 있음.
- 코틀린은 보통 널이 될 수 없는 타입의 값에 대해 널 안전성을 검사하는 연산을 수행하면 경고를 표시하지만
   플랫폼타입의 값은 널 안전성 검사를 중복 수행해도 아무 경고 표시하지 않음.
> [!NOTE]
> **코틀린은 왜 플랫폼 타입을 도입했는가?
> 모든 타입을 널이 될 수 있는 타입으로 다루면 컴파일러가 널 가능성을 판단하지 못하므로 결코 널이 될 수 없는 값에 대해서도 불필요한 null 검사가 들어감
> 코틀린 설계자들은 자바의 타입을 가져온 경우 프로그래머에게 그 타입을 제대로 처리할 책임을 부여하는 실용적인 접근 방법을 택함.
- 플랫폼 타입을 널이 될 수 있는 타입이나 널이 될 수 없는 타입 어느 쪽으로든 사용할 수 있음.
- 우리가 프로퍼티의 널 가능성을 제대로 알고 사용해야 함.





























