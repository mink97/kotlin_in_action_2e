- 코틀린 선언은 기본적으로 final, public
- 내포 클래스는 기본적으로 내부 클래스가 아님. 코틀린 내부 클래스에는 외부 클래스에 대한 암시적 참조가 없다.
- 코틀린 컴파일러는 유용한 메서드를 자동으로 생성
  - data 클래스로 선언하면 컴파일러가 일부 표준 메서드 생성
  - 위임(delegation)을 사용하면 언어가 위임 기능 기본 제공. 위임을 처리하기 위한 준비 메서드를 직접 작성할 필요가 없음.
- object 키워드. 싱글턴 클래스, 동반 객체, 객체식(자바의 익명클래스) 표현 시 사용.
# 1. 클래스 계층 정의

## 코틀린 인터페이스
- 코틀린 인터페이스 안에는 추상 메서드 뿐만 아니라 구현이 있는 메서드도 정의 가능.
- 다만 아무런 상태도 들어갈 수 없음.
- interface 키워드 사용
  `interface Clickable { fun click() }`
- 인터페이스 구현
```kotlin
class Button : Clickable {
  override fun click() = println("I was clicked")
}

fun main() {
  Button.click()
}
```
- 상속이나 구성(인터페이스 구현)에서 모두 클래스 이름 뒤에 콜론을 붙이고 인터페이스나 클래스 이름을 적는 방식 사용
- 인터페이스는 개수 제한 x, 클래스는 하나만 확장 가능
- 오버라이드 시 override 변경자 사용
- 인터페이스 메소드는 디폴트 구현 제공 가능
  - 두 인터페이스에 디폴트 구현이 되어있는 메서드가 있고, 한 클래스에 두 인터페이스를 함께 구현하면 어느 쪽도 선택안함. 오버라이딩 메서드 직접 제공해야 함.
  ```kotlin
  class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")

    override fun showOff() {
      super<Clickable>.showOff()
      super<Focusable>.showOff()
    }
  }
  ```
- 자바 코드는 코틀린의 디폴트 구현을 사용할 수 없음.

## open, final, abstract 변경자 : 기본적으로 final
- 코틀린에서 모든 클래스와 메서드는 기본적으로 final
  - 클래스에 대해 하위 클래스를 만들 수 없고, 기반 클래스의 메서드를 하위 클래스가 오버라이드 할 수 없음.
- **취약한 기반 클래스**(fragile base class)문제 : 기반 클래스 구현을 변경함으로써 하위 클래스가 잘못된 동작을 하는 경우
  -> 특별히 하위 클래스에서 오버라이드하도록 의도된 클래스와 메서드가 아니라면 모두 final
- 어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자 붙이기. 오버라이드를 허용하고 싶은 메서드나 프로퍼티의 앞에도 open
- 기반 클래스나 인터페이스의 멤버를 오버라이드 한 경우 - 기본적으로 open으로 간주
  - 하위 클래스가 오버라이드하는 것을 금지하려면 final로 표시
  - `final override fun click() {}`
- 스마트 캐스트를 사용하려면 프로퍼티가 final이어야 함.
  그렇지 않으면 해당 프로퍼티를 다른 클래스가 상속하면서 커스텀 접근자를 정의함으로써 스마트 캐스트의 요구사항을 깰 수 있음.
- abstract로 선언한 추상 클래스는 인스턴스화할 수 없음.
  - 추상 멤버는 항상 열려있으므로 open 변경자를 명시할 필요가 없음.
  ```kotlin
  abstract class Animated {
    abstract val animationSpeed: Double
    val keyframes: Int = 20
    open val frames: Int = 60

    abstract fun animate()
    open fun stopAnimating() { /* ... */ }
    fun animateTwice() { /* ... */ }
  ```
<img width="1053" alt="image" src="https://github.com/user-attachments/assets/3b4512d4-7a87-4c0a-a940-5f25e3db0271" />
- 인터페이스 멤버는 항상 open(final로 변경 불가). 추상 멤버에 따로 abstract키워드 덧붙일 필요 없음.

## 가시성 변경자: 기본적으로 공개(public)
- public, protected, private
- 모듈 안으로만 한정된 가시성을 제공하기 위해 internal 제공
  - 모듈 구현에 대해 진정한 캡슐화 제공
  - gradle 사용할 때 test 소스 집합은 main 소스의 internal 접근 가능
- 코틀린에서 패키지는 가시성제어에 사용하지 않음.
<img width="1053" alt="image" src="https://github.com/user-attachments/assets/39497905-dacb-42bf-aaa2-5a3dfa7ec2e3" />
- 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 높아야 함.
- 메서드의 시그니처에 사용된 모든 타입의 가시성은 그 메서드의 가시성과 같거나 더 높아야 한다.
- 코틀린의 protected 멤버는 오직 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보임.
- 확장 함수는 해당 클래스의 private이나 protected 멤버에 접근 불가
- 코틀린에서는 외부 클래스가 내부클래스나 내포된 클래스의 private 멤버에 접근할 수 없음.
## 내부 클래스와 내포된 클래스: 기본적으로 내포 클래스
- 클래스 안에 다른 클래스 선언 가능. 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없음.
```kotlin
class Button : View {
  override fun getCurrentState(): State = ButtonState()
  override fun restoreState(state: State) { /*...*/ }
  class ButtonState : State { /*...*/ } // 자바의 static 내포 클래스와 대응
```
- 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자 붙이기
> 내포 클래스 안에는 바깥쪽 클래스에 대한 참조가 없지만 내부 클래스에는 있다.

|클래스 B 안에 정의된 클래스 A|자바에서는|코틀린에서는|
|---|---|---|
|내포 클래스(바깥쪽 참조 저장 x)|static class A|class A|
|내부 클래스(바깥쪽 참조 저장 O)|class A|inner class A|

- 내부 클래스(inner) 안에서 바깥쪽 클래스의 참조에 접근하려면 `this@Outer`라고 써야 함.
  ```kotlin
  class Outer {
    inner class Inner {
      fun getOuterReference(): Outer = this@Outer
    }
  }
  ```
## 봉인된(sealed) 클래스: 확장이 제한된 클래스 계층 정의
- sealed 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스의 가능성 제한 가능
- sealed 클래스의 직접적인 하위 클래스들은 반드시 컴파일 시점에 알려져야 하며, 봉인된 클래스가 정의된 패키지와 같은 패키지에 속해야 하며,
  모든 하위 클래스가 같은 모듈 안에 위치해야 한다.
- sealed 변경자는 클래스가 추상 클래스임을 명시한다.
- 클래스가 아니라 interface 앞에 sealed 변경자를 붙일수도 있음.
  ```kotlin
  sealed interface Toggleable {
    fun toggle()
  }

  class LightSwitch: Toggleable {
    override fun toggle() = println("Lights!")
  }

  class Camera: Toggleable {
    override fun toggle() = println("Camera!")
  }
  ```

# 2. 뻔하지 않은 생성자나 프로퍼티를 갖는 클래스 선언
- 코틀린은 주 생성자와 부 생성자를 구분함.
- 초기화 블록을 통해 초기화 로직을 추가할 수 있음.

## 클래스 초기화: 주 생성자와 초기화 블록
- 클래스 선언
  ```kotlin
  // 1
  class User constructor(_nickname: String) {
    val nickname: String

    init {
      nickname = _nickname
    }
  }

  // 2
  class User(_nickname: String) {
    val nickname = _nickname
  }

  // 3
  class User(val nickname: String)
  ```
- 인스턴스를 만들려면 추가 키워드 없이 생성자를 직접 호출하면 됨.
- 클래스를 정의할 때 별도로 생성자를 정의하지 않으면 컴파일러가 자동으로 디폴트 생성자를 만들어줌.
- 기반 클래스가 아무 인자를 받지 않더라도 해당 클래스를 상속하는 하위 클래스는 반드시 생성자를 호출해야 함. -> 기반 클래스의 이름 뒤에 빈 괄호 필요
  `class RadioButton: Button()`
- 인터페이스는 생성자가 없기 때문에 괄호 없음.
- 어떤 클래스를 클래스 외부에서 인스턴스화하지 못하게 막고 싶다면 생성자를 private으로 만들어야 한다.
  `class Secretive private constructor(private val agentName: String) {}`

## 부 생성자: 상위 클래스를 다른 방식으로 초기화
> 인자에 대한 기본값을 제공하기 위해 부 생성자를 여럿 만들지 말라. 대신 파라미터의 기본값을 생성자 시그니처에 직접 명시하라.
- 부 생성자는 **constructor** 키워드로 시작
  ```kotlin
  open class Downloader {
    constructor(url: String?) {
      // 어떤 코드
    }

    constructor(uri: URI?) {
      // 어떤 코드
    }
  }

  class MyDownloader : Downloader {
    constructor(url: String?) : super(url) {
      //...
    }

    constructor(uri: URI?) : super(uri) {
      //...
    }
  }

  class MyDownloader2 : Downloader {
    constructor(url: String?) : this(URI(url)) // 같은 클래스의 다른 생성자에게 위임
    constructor(uri: URI?) : super(uri)
  }
  ```
- 클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야함.

## 인터페이스에 선언된 프로퍼티 구현
- 코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.
  ```kotlin
  interface User {
  	val nickname: String
  }
  
  class PrivateUser(override val nickname: String) : User
  
  class SubscribingUser(val email: String) : User {
      override val nickname: String
      	get() = email.substringBefore('@')
  }
  
  class SocialUser(val accountId: Int) : User {
      override val nickname = getNameFromSocialNetwork(accountId)
  }
  
  fun getNameFromSocialNetwork(accountId: Int) = "kodee$accountId"
  
  fun main() {
      println(PrivateUser("kodee").nickname)
      
      println(SubscribingUser("test@kotlinlang.org").nickname)
      
      println(SocialUser(123).nickname)
  }
  ```
- SubscribingUser의 닉네임은 호출될 때마다 substringBefore를 호출해 계산 / SocialUser는 객체 초기화 시 계산한 데이터를 필드에 저장했다가 불러오는 방식
- 인터페이스에 추상 프로퍼티뿐 아니라 게터와 세터가 있는 프로퍼티 선언 가능(뒷받침하는 필드 참조 불가)
> ### 함수 대신 프로퍼티를 사용하는 경우
> - 예외를 던지지 않는다.
> - 계산 비용이 적게 든다.
> - 객체 상태가 바뀌지 않으면 여러 번 호출해도 항상 같은 결과를 돌려준다.

## 게터와 세터에서 뒷받침하는 필드에 접근
- 값을 저장하는 동시에 로직을 실행할 수 있게 하려면 접근자 안에서 프로퍼티를 뒷받침하는 필드에 접근할 수 있어야 함.
- 세터에서 뒷받침하는 필드 접근
  ```kotlin
  class User(val name: String) {
    var address: String = "unspecified"
      set(value: String) {
        println(
          """
          Address was changed for $name:
          "$field" -> "$value".
          """.trimIndent())
      field = value
    }
  }

  fun main() {
    val user = User("Alice")
    user.address = "Christoph-Rapparini-Bogen 23"
  }
  ```
- 게터에서는 field 값을 읽을 수만 있고, 세터에서는 field 값을 읽거나 쓸 수 있다.
- 변경 가능 프로퍼티의 게터와 세터 중 한쪽만 직접 정의해도 됨.
- 컴파일러는 게터나 세터에서 field를 사용하는 프로퍼티에 대해 뒷받침하는 필드를 생성. 사용하지 않으면 생성x

## 접근자의 가시성 변경
- 접근자의 가시성은 프로퍼티의 가시성과 같지만 get이나 set앞에 가시성 변경자를 추가하여 접근자의 가시성 변경 가능
  ```kotlin
  class LengthCounter {
    var counter: Int = 0
      private set

    fun addWord(word: String) {
      counter += word.length
    }
  }
  ```

# 3. 컴파일러가 생성한 메서드: 데이터 클래스와 데이터 위임
## 모든 클래스가 정의해야 하는 메서드
- toString, equals, hashCode
## 데이터 클래스: 위의 메서드를 자동으로 생성
- 어떤 클래스가 데이터를 저장하는 역할만을 수행한다면 toString, equals, hashCode를 반드시 오버라이드 해야 함.
- data라는 변경자를 클래스 앞에 붙이면 컴파일러가 자동으로 생성.(데이터 클래스)
- 주 생성자 밖에서 정의된 프로퍼티는 equals나 hashCode를 계산할 때 고려의 대상이 아님.
### 데이터 클래스와 불변성: copy() 메서드
- 데이터 클래스는 위 메서드들 외에도 다른 편의 메서드 제공(ex. copy())
- 데이터 클래스의 프로퍼티가 꼭 val일 필요는 없지만 권장.
  - hashMap 등의 컨테이너에 데이터 클래스 객체를 담는 경우, 불변성 필수
- 불변 객체를 사용하면 프로그램에 대해 훨씬 쉽게 추론 가능.(다중 스레드 프로그램의 경우 더 중요, 사용중인 데이터를 다른 스레드가 변경할지 신경 쓸 필요 x)
- copy() : 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해줌
  ```kotlin
  fun copy(name: String = this.name,
      postalCode: Int = this.postalCode) =
    Customer(name, postalCode)
  ```
## 클래스 위임: by 키워드 사용
- 종종 상속을 허용하지 않는 클래스에게 새로운 동작을 추가해야 할 때가 있음 -> **데코레이터 패턴** 사용
- 상속을 허용하지 않는 기존 클래스 대신 사용할 수 있는 새로운 클래스를 만들되, 기존 클래스와 같은 인터페이스를 데코레이터가 제공하고 기존 클래스를 데코레이터 내부 필드로 유지
- 새로운 기능은 새로 정의하고, 기존 기능이 그대로 필요한 부분은 데코레이터의 메서드가 기존 클래스의 메서드에게 요청 전달
- 코틀린에서 인터페이스를 구현할 때 by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.
  ```kotlin
  class CountingSet<T>(
  	private val innerSet: MutableCollection<T> = hashSetOf<T>()
  ) : MutableCollection<T> by innerSet {
      
      var objectsAdded = 0
      
      override fun add(element: T): Boolean {
          objectsAdded++
          return innerSet.add(element)
      }
      
      override fun addAll(elements: Collection<T>): Boolean {
          objectsAdded += elements.size
          return innerSet.addAll(elements)
      }
  }
  
  fun main() {
      val cset = CountingSet<Int>()
      cset.addAll(listOf(1, 1, 2))
      println("Added ${cset.objectsAdded} objects, ${cset.size} uniques.")
  }
  ```
- 위임하면 위임 대상 내부 클래스에 문서화된 API를 활용해 기능을 구현. 내부 클래스가 문서화된 API를 변경하지 않는 한 코드가 계속 잘 작동할 것임을 확신

# 4. object 키워드: 클래스 선언과 인스턴스 생성을 한꺼번에 하기
- object 키워드를 사용하는 여러 상황
  - **객체 선언**: 싱글턴을 정의하는 한 가지 방법
  - **동반 객체**: 어떤 클래스와 관련이 있지만 호출하기 위해 그 클래스의 객체가 필요하지는 않은 메서드와 팩토리 메서드를 담을 때 쓰임.
          동반 객체의 멤버에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사용.
  - **객체 식**: 자바의 익명 내부 클래스 대신 사용
## 객체 선언: 싱글턴을 쉽게 만들기
- 코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원.
- 객체 선언 : 클래스 선언 + 단일 인스턴스의 선언을 합친 선언.
```kotlin
object Payroll {
  val allEmployees = arrayListOf<Person>()

  fun calculateSalary() {
    for (person in allEmployees) {
      /* ... */
    }
  }
}
```
- 객체 선언은 클래스를 정의하고 그 클래스의 인스턴스를 만들어 변수에 저장하는 모든 작업을 한 문장으로 처리
- 객체 선언에서는 생성자를 쓸 수 없음.(객체 선언 시 즉시 생성되므로 생성자 정의가 필요없음)
- 제공하고 싶은 초기 상태가 있다면 그 객체의 본문에서 직접 제공
- 프레임워크를 사용하기 위해 특정 인터페이스를 구현해야하는데, 구현 내부에 다른 상태가 필요하지 않은 경우 유용(이부분이 조금 불안)
- 객체 선언을 사용해 Comparator 구현하기
  ```kotlin
  import java.io.*
  
  object CaseInsensitiveFileComparator : Comparator<File> {
      override fun compare(file1: File, file2: File): Int {
          return file1.path.compareTo(file2.path, ignoreCase = true)
      }
  }
  
  fun main() {
      val files = listOf(File("/Z"), File("/a"))
      println(files.sortedWith(CaseInsensitiveFileComparator))
      println(
      	CaseInsensitiveFileComparator.compare(
          	File("/User"), File("/user")
          )
      )
  }
  ```
- 클래스 안에서 객체(object) 선언도 가능(해당 객체도 하나)
- 내포 객체를 사용해 Comparator 구현하기
  
## 동반 객체: 팩토리 메서드와 정적 멤버가 들어갈 장소
- 코틀린 클래스 안에는 정적인 멤버가 없음.(코틀린 언어는 자바 static 키워드를 지원하지 않음.)
- 패키지 수준의 최상위 함수와 객체 선언을 활용(대부분의 경우 최상위 함수 활용을 권장)
- 클래스의 인스턴스와 관계없이 호출해야 하지만, 클래스 내부 정보에 접근해야 하는 함수가 필요할 때!
  클래스에 내포된 객체 선언의 멤버 함수로 정의 가능
- 클래스 안에 정의된 object 중 하나에 companion이라는 표시를 붙이면 자신을 감싸는 클래스의 이름을 통해 직접 사용가능
  ```kotlin
  class MyClass {
    companion object {
      fun callMe() {
        println("Companion object called")
      }
    }
  }

  fun main() {
    MyClass.callMe()
    val myObject = MyClass()
    // myObject.callMe() // error
  }
  ```
- 해당 클래스의 인스턴스는 companion object에 접근할 수 없다.
- 동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근 가능 -> [팩토리 패턴](https://ko.wikipedia.org/wiki/%ED%8C%A9%ED%86%A0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C_%ED%8C%A8%ED%84%B4)을 구현하기 가장 적합한 위치
```kotlin
class User private constructor(val nickname: String) {
  companion object {
    fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
    fun newSocialUser(accountId: Int) = User(getNameFromSocialNetwork(accountId))
  }
}

fun main() {
  val subscribingUser = User.newSubscribingUser("bob@gmail.com")
  val socialUser = User.newSocialUser(4)
  println(subscribingUser.nickname)
}
```
- 팩토리 메서드는 생성할 필요가 없는 객체를 생성하지 않을 수 있음.
  - 예를 들어 이메일 주소별로 유일한 User 인스턴스를 만드는 경우 팩토리 메서드가 이미 존재하는 인스턴스에 해당하는 이메일 주소를 전달받으면 기존 인스턴스 반환
  - 클래스를 확장해야 하는 경우 동반 객체 멤버를 하위 클래스에서 오버라이드할 수 없으므로 여러 생성자를 사용하는 편이 낫다.(이부분도 조금 불안)

## 동반 객체를 일반 객체처럼 사용
- 동반 객체도 이름 붙이거나(클래스 이름으로 동반 객체 내 멤버 참조 가능, 이름 짓느라 고생할 필요X),<br/>인터페이스를 상속하거나,   동반 객체 안에 확장함수와 프로퍼티를 정의 가능
- 동반 객체 이름 지정하지 않으면 자동으로 Companion이 됨.
- 싱글턴 동반 객체를 코틀린 표준 라이브러리에서도 볼 수 있음.(Random클래스 내에 companion object Default, 기본 난수 생성기 제공)
  ```kotlin
  val chance = Random.nextInt(from=0, until=100)
  val coin = Random.Default.nextBoolean()
  ```
### 동반 객체에서 인터페이스 구현
```kotlin
interface JSONFactory<T> {
  fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
  companion object : JSONFactory<Person> {
    override fun fromJSON(jsonText: String): Person = /* ... */
  }
}

fun <T> loadFromJson(factory: JSONFactory<T>): T {
  /* ... */
}

loadFromJSON(Person) // JSONFactory넘길때 Person클래스 이름 사용
```

### 동반 객체 확장
- 클래스에 동반 객체가 있으면 그 객체 안에 함수를 정의함으로써 클래스에 대해 호출할 수 있는 확장 함수를 만들 수 있다.
```kotlin
class Person(val firstName: String, val lastName: String) {
  companion object {
  }
}

fun Person.Companion.fromJSON(json: String): Person { // 확장 함수 선언

}

val p = Person.fromJSON(json)
```
- 동반 객체에 대한 확장 함수를 작성할 수 있으려면 원래 클래스에 동반 객체를 꼭 선언해야 함.(빈 객체라도)

## 객체 식: 익명 내부 클래스를 다른 방식으로 작성
- 익명 객체 정의할 때 object사용
  ```kotlin
  interface MouseListener {
    fun onEnter()
    fun onClick()
  }

  class Button(private val listener: MouseListener) { /* ... */ }
  ```
- 객체 식을 사용하면 임의의 MouseListener 구현을 생성해서 Button 생성자에게 넘길 수 있음.
  ```kotlin
  fun main() {
    Button(object : MouseListener {
      override fun onEnter() { /* ... */ }
      override fun onClick() { /* ... */ }
    })
  }
  ```
- 객체 식은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만 그 클래스나 인스턴스에 이름을 붙이지는 않는다.
- 익명 객체는 인터페이스를 하나 구현할 수도, 여럿 구현할 수도, 구현하지 않을 수도 있음.
- 자바의 익명 클래스처럼 식이 포함된 함수의 변수에 접근 가능 + final이 아니어도 사용가능 -> 변수 값 변경도 가능
  ```kotlin
  fun main() {
    var clickCount = 0 // 로컬 변수 정의
    Button(object : MouseListener {
      override fun onEnter() { /* ... */ }
      override fun onClick() {
        clickCount++  // 로컬 변수 값 변경
      }
    })
  }
  ```
- 객체 식은 익명 객체 안에서 여러 메서드를 오버라이드해야 하는 경우에 훨씬 더 유용(메서드가 하나라면 SAM변환, 람다 이용)
# 5. 부가 비용 없이 타입 안정성 추가: 인라인 클래스
- 함수에 값을 전달할 때 서로 다른 의미의 값을 전달하는 것을 막기 위해 기존 타입 대신 클래스 사용
  - 함수에 잘못된 의미의 값을 전달할 위험이 줄어들지만 함수 호출 때마다 객체를 생성하기 때문에 성능 상 불리
  - 인라인 클래스 사용
  ```kotlin
  @JvmInline
  value class UsdCent(val amount: Int)
  ```
- 실행 시점에 인라인 클래스를 가능한 최대한 내부 프로퍼티의 타입으로 표현. 클래스의 데이터가 사용되는 지점에 인라인
- 인라인으로 표현하려면 클래스가 프로퍼티를 하나만 가져야 하며, 주 생성자에서 초기화
- 인라인 클래스는 다른 클래스를 상속할 수도 없고, 다른 클래스가 상속할 수도 없음.
- 인터페이스를 상속하거나 메서드를 정의하거나, 계산된 프로퍼티 제공은 가능
  ```kotlin
  interface PrettyPrintable {
    fun prettyPrint()
  }

  @JvmInline
  value class UsdCent(val amount: Int): PrettyPrintable {
    val salesTax get() = amount * 0.06
    override fun prettyPrint() = println("${amount}¢")
  }

  fun main() {
    val expense = UsdCent(1_99)
    println(expense.salesTax)
    expense.prettyPrint()
  }
# 요약
- 코틀린의 인터페이스는 자바 인터페이스와 비슷하지만 **디폴트 구현**과 **프로퍼티**도 포함할 수 있다.
- 모든 코틀린 선언은 기본적으로 final이며 public이다.
- 상속, 오버라이딩이 가능하게 하려면 앞에 open을 붙여야 한다.
- internal 선언은 같은 모듈 안에서만 볼 수 있다.
- 내포 클래스는 기본적으로 내부 클래스가 아니다. 바깥쪽 클래스에 대한 참조를 내포 클래스 안에 포함시키려면 **inner** 키워드를
  클래스 선언 앞에 붙여야 한다.
- sealed 클래스를 직접 상속하는 클래스나 sealed 인터페이스에 대한 구현들은 모두 컴파일 시점에 컴파일러에 보여야 한다.
- 초기화 블록과 부 생성자를 활용해 클래스 인스턴스를 더 유연하게 초기화할 수 있다.
- field 식별자를 통해 게터와 세터 안에서 프로퍼티의 데이터를 저장하는 데 뒷받침하는 필드를 참조할 수 있다.
- 데이터 클래스를 사용하면 컴파일러가 equals, hashCode, toString, copy 등의 메서드를 자동으로 생성해준다.
- 클래스 위임을 사용하면 위임 패턴을 구현할 때 비슷한 위임 코드를 반복하는 것을 피할 수 있다.
- 객체 선언은 싱글턴 클래스를 정의하는 코틀린식 방법
- 동반 객체는 자바의 정적 메서드와 필드 정의를 대신한다.
- 동반 객체도 다른 객체와 마찬가지로 인터페이스를 구현할 수 있으며, 동반 객체에 대해서도 확장 함수와 프로퍼티를 정의할 수 있다.
- 코틀린의 객체 식은 자바의 익명 내부 클래스를 대신하며, 여러 인터페이스를 구현하거나
  객체가 포함된 영역에 있는 변수의 값을 변경할 수 있는 등 자바 익명 내부 클래스보다 더 많은 기능을 제공한다.
- 인라인 클래스를 사용하면 수명이 짧은 객체를 많이 할당함으로써 생길 수 있는 성능 저하를 피하면서 새로운 타입 안정성 계층 추가 가능하다.

# 중요하게 생각한 점.
- 클래스에 대한 얘기가 많다보니 관련한 여러 개념들과 용어들이 많이 나와서 잘 숙지하고 가야겠네...
- `interface` : 인터페이스 정의
  - 디폴트 구현, 프로퍼티 포함 가능
- `override` : 오버라이드 변경자
- `open` / `final` / `abstract` 변경자
- 가시성 변경자 : `public` / `protected`(하위 클래스에서만) / `private` / `internal`(모듈 안에서만)
- `inner` : 내포 클래스를 내부클래스로 변경. 바깥쪽 클래스에 대한 참조 가능
- `sealed` : 하위 클래스들 컴파일 시점에 알려지고, 같은 패키지, 같은 모듈 내에 위치. 인터페이스에도 가능
- 클래스 초기화 : 주 생성자, 부 생성자
- `field` : 뒷받침하는 필드에 접근(게터, 세터)
- 접근자의 가시성 변경가능
- `data class` : 데이터 클래스
- `by` : 클래스 위임
- `object` : 객체 선언 / 객체 식(익명 객체)
- `companion object` : 동반 객체, 정적멤버(특히 private멤버에 접근 시), 팩토리 메서드
- `@JvmInline`, `value class` : 인라인 클래스







