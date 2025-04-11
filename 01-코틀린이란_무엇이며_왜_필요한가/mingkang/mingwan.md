# 1. 코틀린 맛보기
- 클래스 본문 지정 없이 프로퍼티가 포함된 Person 데이터 클래스 정의
- val 키워드를 사용해 읽기 전용 프로퍼티 선언
- 인자의 기본값 제공
- 널이 될 수 있는 값들을 명시적으로 다룸 -> NullPointerException 방지
- 클래스 안에 포함될 필요가 없는 최상위 함수 정의
- 함수나 생성자 호출 시 파라미터 이름을 지정하여 사용 가능
- [트레일링 콤마](https://femiwiki.com/w/%ED%8A%B8%EB%A0%88%EC%9D%BC%EB%A7%81_%EC%BD%A4%EB%A7%88)
를 사용한다
- 람다식을 사용해 컬렉션 연산을 사용한다
- 엘비스연산자(?:) 사용, null일 때 대비
- 문자열 템플릿을 사용한다
- 데이터 클래스의 toString 자동으로 생성
```
data class Person(
  val name: String,
  val age: Int? = null
)

fun main() {
  val persons = listOf(
    Person("영희", age = 29),
    Person("철수"),
  )

  val oldest = persons.maxBy {
    it.age ?: 0
  }
  println("가장 나이가 많은 사람: $oldest")
}
// 가장 나이가 많은 사람 : Person(name=영희, age=29)
```
<br />

# 2. 코틀린의 주요 특성
## 1. 용례
  - 안드로이드 디바이스에서 실행되는 모바일 애플리케이션
  - 서버상의 코드(특히 웹 애플리케이션의 백엔드)
  - KMP(코틀린 멀티플랫폼)를 이용하여 다른 환경의 애플리케이션도 만들 수 있음.
## 2. 정적타입지정 언어
  - **성능** : 메소드 호출이 더 빠름.
  - **신뢰성** : 실행 시점에 프로그램이 실패할 가능성 줄어듬.
  - **유지보수성** : 코드가 작용하는 타입을 볼 수 있어서(타입 추론 안했을 때인 듯)
  - **도구지원** : 컴파일 시점에 오류 많이 잡음.
- 코틀린은 타입 추론을 제공
- nallable type 지원 -> null pointer exception여부를 컴파일 시점에 검사, 신뢰성 ⬆️
## 3. 함수형 프로그래밍 + 객체지향 프로그래밍
  - 함수형 프로그래밍 특징
    1. 함수를 일반 값처럼 다룸(변수에 저장, 파라미터로 함수를 넣거나 함수를 리턴)
    2. 불변성 : 불변객체를 사용하여 프로그램 작성
    3. 부수효과 없음 : 입력 같으면 항상 같은 출력, 다른 객체의 상태 변경 X, 외부의 다른 환경과 상호작용 X
  - 함수형 프로그래밍 장점
    1. 코드 중복 피할 수 있음
    2. 안전한 동시성(안전하지 않은 데이터 변경이 발생하지 않음을 확신할 수 있음)
    3. 테스트 쉬움 : 부수효과가 없기 때문
  ```
  messages
    .filter { it.body.isNotBlank() && !it.isRead }
    .map(Message::sender)
    .distinct()
    .sortedBy(Serner::name)
  ```
  - 코틀린은 객체지향과 함수형 접근 방식을 조합해서 사용가능
    - 함수형 스타일 + 필요할 땐 가변 데이터, 부수 효과 사용
## 4. 코루틴을 통해 동시성, 비동기 코드를 자연스럽고 구조적으로 사용 가능
<br />

# 3. 코틀린이 자주 쓰이는 분야
## 1. 백엔드 지원
```
@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
  runApplication<DemoApplication>(*args)
}

@RestController
class GreetingResource {
  @GetMapping
  fun index(): List<Greeting> = listof(
    Greeting(1, "Hello!"),
    Greeting(2, "Bonjour!"),
    Greeting(3, "Guten Tag!"),
  )
}

data class Greeting(val id: Int, val text: String)
```
- 스프링 외에도 Ktor, http4k같은 코틀린 서버 프레임워크들도 있음
## 2. 모바일 개발: 안드로이드는 코틀린 우선
  - KTX 라이브러리, Jetpack Compose...
  ```
  // Jetpack Compose 예시
  @Composable
  fun MessageCard(modifier: Modifier, message: Message) {
    var isExpanded by remember { mutableStateOf(false) } // 상태 기억
    Column(modifier.clickable { isExpanded = !isExpanded }) {
      Text(message.body)
      if (isExpanded) {
        MessageDetails(message)
      }
    }
  }

  @Composable
  fun MessageDetails(message: Message) { /* ... */ }
  ```
## 3. 다중 플랫폼
<br />
   
# 4. 코틀린의 철학
## 1. 코틀린은 실용적인 언어다
   - 실제 문제를 해결하기위한 실용적 언어
   - 코틀린의 경우 인텔리제이 IDEA 플러그인의 개발과 컴파일러의 개발이 맞물려있음.(편리한 개발환경, 도구 강조)
## 2. 코틀린은 간결하다
   - 코드가 간결할수록 내용 파악 쉬움
   - getter, setter, 생성자 파라미터 들 객체지향 언어의 여러 가지 준비 코드를 코틀린은 암시적으로 제공.
   - 타입 추론 제공
   - 풍부한 표준 라이브러리
## 3. 코틀린은 안전하다
   - JVM에서 실행되는 정적타입지정 언어 = 타입 안전성 보장
   - val(읽기 전용), data class 등을 통해 다중 스레드 환경에서 안전성을 더 보장
   - null체크에 신경을 씀.(타입 이름 끝에 ? 추가하면 널이 될 수 있음을 의미)
   - 코틀린은 타입 검사와 캐스트를 한 연산자로 수행. 타입 검사를 했으면 해당 타입의 메서드 사용가능(스마트 캐스트)
     ```
     fun modify(value: Any) {
       if (value is String) {
         println(value.uppercase())
       }
     }
     ```
## 4. 코틀린은 상호 운용성이 좋다
   - 기존 라이브러리 사용 가능
   - 자바와 코틀린 코드를 프로젝트에서 원하는 대로 섞어 쓸 수 있음.
# 요약
- 코틀린은 타입 추론을 지원하는 정적타입지정 언어
- 객체지향과 함수형 프로그래밍 스타일 모두 지원
- 코루틴은 경량 스레드. 비동기 코드를 순차적 코드와 비슷한 로직으로 작성 가능.

# 문제
## Q1. 코틀린 맛보기에서 나온 설명과 코드를 비교해보기
## Q2. 다음 코드 해석하기
  ```
  messages
    .filter { it.body.isNotBlank() && !it.isRead }
    .map(Message::sender)
    .distinct()
    .sortedBy(Serner::name)
  ```

## A2.
- 읽지 않고 비어있지 않은 메시지를 찾아 송신자들의 모든 고유한 이름을 정렬해
