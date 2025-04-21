# 클래스 계층 정의

## 코틀린 인터페이스

```kotlin
fun main() {
    Button().click() // I was Clicked
    Button().showOff() // I'm Clickable!
}

interface Clickable {
    fun click() // 추상 메소드
    fun showOff() = println("I'm Clickable!")
}

class Button: Clickable {
    override fun click() = println("I was Clicked")
}
```

- 자바랑 달리 인터페이스는 interface로 선언.
- 인터페이스에는 상태가 들어갈 수 없다.
- 인터페이스 안에 추상 메소드 뿐 아니라 구현이 있는 메소드가 들어갈 수 있음.
- 자바와 마찬가지로 인터페이스 구현과 상속 모두 콜론을 사용.
- `override 변경자를 꼭 사용`해야한다는 점이 자바와의 차이.
- default 구현 메소드를 제공할 수 있음.

<aside>
💡

인터페이스 함수는 Unit 타입을 반환한다.

Unit은 자바로 치면 void에 대응된다.

</aside>

```kotlin
interface Clickable {
    fun click() // 추상 메소드
    fun showOff() = println("I'm Clickable!")
}

class Button: Clickable, Focusable { // 같은 메소드를 구현하는 다른 인터페이스 정의
    override fun click() = println("I was Clicked")
}

interface Focusable {
    fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm Focusable!")
}
```

- 어떤 클래스가 같은 메소드를 구현하는 서로 다른 2개 이상의 인터페이스를 구현할 수 없음. 이 경우 컴파일 오류 발생.

```kotlin
fun main() {
    val button = Button()
    button.showOff() // I'm Clickable! I'm Focusable!
    button.setFocus(true) // I got focus.
    button.click() // I was Clicked
}

class Button: Clickable, Focusable {
    override fun click() = println("I was Clicked")

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

- 이 경우 명시적으로 새로운 구현을 제공하면 해결된다.
- 상위 타입의 메소드는 `super<인터페이스명>.함수명()` 형식으로 호출할 수 있다.

```kotlin
public class JavaButton implements Clickable {
    @Override
    public void click() {
        Clickable.DefaultImpls.click(this);  // 보조 클래스의 static 메서드 호출
    }

    @Override
    public void showOff() {
        Clickable.DefaultImpls.showOff(this);
    }
}
```

- 아까 코드를 자바에서 쓰려면 이런식으로 코틀린에서 메소드 본문을 제공하는 메소드를 포함하는 모든 메소드에 대한 본문을 작성해줘야 한다.
- showOff가 default 구현을 제공해주지만, 명시적으로 showOff를 override하여, Clickable.DefaultImpls.showOff(this); 를 호출해준 모습이다.

## class는 기본적으로 final 하다.

- 코틀린은 클래스에 대해 하위 클래스를 만들 수 없고, base 클래스의 메소드를 sub 클래스가 override 할 수 없다. 즉, final하다.
- 자바에서는 명시적으로 final 키워드를 붙이지 않으면 final 하지 않다.
- 이펙티브 자바의 [***상속을 고려해 설계하고 문서화하라 그러지 않았다면 상속을 금지하라***](https://cabi.oopy.io/e1f8c3d4-4942-4f87-be50-0b3f0af8b483)를 참조하자.
- 코틀린도 이런 자바의 철학을 존중하여 한 발 더 나아가 final을 default로 잡은 것이다.

```kotlin
open class RichButton: Clickable {
    fun disable() = println("disabled")
    open fun animate() = println("animate")
    override fun click() {
    }
}

class SuperRichButton: RichButton {
    override fun disable() = println("disabled")
    override fun animate() {
        super.animate()
    }
    override fun click() {
        super.click()
    }

    override fun showOff() {
        super.showOff()
    }
}
```

- RichButton은 열린 클래스다. 다른 클래스가 상속할 수 있다.
- disable은 final한 메소드이다. 이 메소드를 override 할 수 없다.
    - disable을 상속 시도하면 컴파일 에러가 발생한다.
- animate는 open 메소드이다. 이 메소드를 override 할 수 있다.
- RichButton이 showOff을 override 하지 않았어도 SuperRichButton 에서 override 가능하다.

<aside>
💡

클래스가 기본적으로 final 함에 따라, 스마트 캐스트를 용이하게 할 수 있다.

예를 들어 클래스 프로퍼티가 final 이 아니라면, 그 프로퍼티를 다른 클래스가 상속하면서 커스텀 접근자를 정의할 수 있게 되어, 스마트 캐스트가 불가능해지는 상황이 발생할 수 있다. (스마트 캐스트는 프로퍼티가 val이면서 커스텀 접근자가 없는 경우에만 사용할 수 있음.)

</aside>

```kotlin
abstract class Animated {
    abstract val animationSpeed: Double
    val keyframes: Int = 20
    open val frames: Int = 60

    abstract fun animate()
    open fun stopAnimating() {}
    fun animateTwice() {}
}
```

- Animated는 추상 클래스이므로 인스턴스화 불가능.
- 추상 프로퍼티는 값이 없고, 하위 클래스는 반드시 값이나 접근자를 제공해야 한다.
- 마찬가지로 추상 클래스의 추상이 아닌 프로퍼티는 기본 final이며 open 가능하다.
- animate는 추상 함수이며 반드시 상위 클래스에서 구현해야 한다.
- 추상 클래스의 추상이 아닌 함수는 기본 final이며 open 가능하다.

## 가시성 변경자는 기본적으로 open 하다.

- 가시성 변경자는 public, protected, private 변경자를 말한다.
- 자바와 동일하다.
- 대신에 `모듈 안으로 한정`된 가시성인 `internal`이라는 가시성을 제공한다.
- 패키지 전용 가시성은 없다. (ex. package-private)
    - 코틀린의 패키지는 네임스페이스 관리 목적만 가진다. (가시성 제어에 활용 x)
    - 캡슐화 제공은 internal로 퉁친다는 느낌같다. (interal이 더 캡슐화를 지킨다고 생각하는 것 같다.)

```kotlin
internal open class TalkativeButton {
    private fun yell() = println("Hey!")             
    protected fun whisper() = println("Let's talk!") 
}

// public 확장 함수
fun TalkativeButton.giveSpeech() { 
    yell()    // 오류 ❌: private 라서 못 씀
    whisper() // 오류 ❌: protected 라서 못 씀
}
```

- public 멤버가 자신의 internal 수신 타입을 노출시켜버림.
    - `TalkativeButton`은 internal → "같은 모듈 안에서만 접근 가능"
    - `giveSpeech()`는 public → "어디서나 호출 가능해야 함"
    - 두 전제가 충돌하며 혼돈의 카오스로 인해 컴파일 에러 발생.
        
        ![image.png](attachment:8c31a768-10e7-4cb0-8a1d-102304c835a4:image.png)
        

**해결 방법**

1. giveSpeech의 가시성을 private이나 internal로 바꾼다. (어쨋든 public보다 좁게)
2. TalkativeButton의 가시성을 public으로 바꾼다.

<aside>
💡

1. 코틀린의 `public`, `protected`, `private` 가시성은 자바 바이트코드에서도 그대로 유지된다.
2. 단, 코틀린의 `private class`는 자바에서 허용되지 않아 **패키지 전용(package-private)**으로 컴파일된다.
3. `internal`은 자바에 대응되는 개념이 없어, 바이트코드상에서는 **`public`으로 처리된다.**
4. 따라서 자바 코드에서는 코틀린에서 접근 불가능한 `internal` 요소에 접근할 수 있는 경우가 생긴다.
5. 이를 방지하려고 코틀린 컴파일러는 `internal` 멤버의 이름을 **보기 어렵게 변형**해 실수로 쓰는 걸 막는다.
</aside>

## inner 클래스 & nested 클래스: 기본값 nested 클래스

- 자바처럼 헬퍼 클래스 캡슐화하고 싶을 때, nested 클래스 쓸 수 있다.
- 그런데 자바랑 다르게 inner 에서 outer 객체 참조 못한다.

### 결론부터

| 용어 | 자바 | 코틀린 |
| --- | --- | --- |
| **내포 클래스**(Nested Class) | `static`으로 선언해야 outer class 참조 안 함 | **기본이 static(outer 참조 X)** |
| **내부 클래스**(Inner Class) | 기본이 inner 클래스 (outer 참조 포함됨) | `inner` 키워드를 써야 outer 참조 포함됨 |

```java
public class Button implements View {
    @Override
    public State getCurrentState() {
        return new ButtonState(); // 내부 클래스
    }

    public class ButtonState implements State, Serializable {
        // ButtonState는 암시적으로 Button 인스턴스를 참조함
    }
}
```

- `ButtonState`는 inner 클래스이기 때문에 `Button` 인스턴스를 참조함
    - 자바는 inner 가 outer 인스턴스를 암시적으로 참조함.
- 직렬화하려 하면 `Button`도 같이 직렬화해야 하는데, `Button`이 `Serializable`이 아님
    
    → 그래서 `NotSerializableException: Button` 발생
    

```java
public class Button implements View {
    @Override
    public State getCurrentState() {
        return new ButtonState(); // 내부 클래스
    }

    public static class ButtonState implements State, Serializable {
        // 외부 참조가 없어져 직렬화 SSAP 가능
    }
}
```

- `ButtonState`를 `static class`로 바꾸면 외부 참조가 없어져서 직렬화 가능해짐

```kotlin
class Button : View {
    override fun getCurrentState(): State = ButtonState()

    class ButtonState : State // 기본이 static이라서 외부(Button)를 참조하지 않음
}
```

- 킹틀린에서는 `ButtonState`는 바깥 `Button`을 참조하지 않음
    - inner에서 outer에 대한 암시적 참조가 일어나지 않음.
- 자바의 `static class`처럼 동작함 → 직렬화 SSAP 가능

```kotlin
class Button : View {
    override fun getCurrentState(): State = ButtonState()

    inner class ButtonState : State { // inner로 정의하여, outer 참조 가능
        fun getOuterReference(): Button = this@Button // outer를 실제로 참조하는 방법
    }
}
```

- 다만, 명시적으로 inner 에서 outer 인스턴스를 참조하게 만들고 싶다면 ?
- nested class 앞에 inner 변경자를 붙여야 한다~

## sealed: 확장이 제한된 클래스 계층 정의 방법

```kotlin
interface Expr

class Num(val value: Int) : Expr

class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int = when (e) {
    is Num -> e.value
    is Sum -> eval(e.left) + eval(e.right)
    else -> throw IllegalArgumentException("Unknown expression")
}

```

- 매 번 else 분기 쓰는게 불편하다!
- 괜한 실수로 인해 심각한 버그 발생 가능성이 있는게 짜증나다!

```kotlin
sealed class Expr

class Num(val value: Int) : Expr()
class Sum(val left: Expr, val right: Expr) : Expr()

fun eval(e: Expr): Int = when (e) {
    is Num -> e.value
    is Sum -> eval(e.left) + eval(e.right)
}
```

- interface 대신에 sealed 클래스를 사용해보자.
    - sealed를 붙이면 추상 클래스로 인식된다.
- sealed 클래스를 직접 상속한 하위 클래스가 컴파일 시점에 알려지기만 한다면,
- else 식으로 예외 처리하지 않고 깔끔하게 처리할 수 있다.

```kotlin
sealed class Expr

class Num(val value: Int) : Expr()
class Sum(val left: Expr, val right: Expr) : Expr()
class Mul(val left: Expr, val right: Expr): Expr()

fun eval(e: Expr): Int = when (e) {
    is Num -> e.value
    is Sum -> eval(e.left) + eval(e.right)
}
```

- Mul을 정의해두고, eval의 when 절에서 확인하지 않으면,
- `'when' expression must be exhaustive, add necessary 'is Mul' branch or 'else' branch instead`
    - 이런식으로 when절 뽀갈난다고 컴파일 타임에 미리 알려준다.
- 오류 발생 가능성이 있는 코드를 런타임이 아닌 컴파일 타임에 미리 알 수 있는 코드 스타일이므로 적극 사용하면 좋겠다!

## fun 하지 않은 생성자나 프로퍼티를 갖는 클래스

- 자바와 마찬가지로 생성자를 여러개 정의할 수 있다.
- 대신, 주 생성자와 부 생성자로 구분된다.

### 주 생성자 초기화 블록

```kotlin
class User(val nickname: String)
```

- 클래스 이름 뒤에 괄호로 둘러싸인 코드 == `주 생성자`
- nickname 이라는 프로퍼티도 동시에 정의 및 초기화함.

```kotlin
class User constructor(_nickname: String) {
    val nickname: String

    init {
        nickname = _nickname
    }
}
```

- 주 생성자의 실제 풀버전은 다음과 같음.

```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true)
```

- 이전에 나온 것 처럼 생성자에도 default 인자 지정 가능.

```kotlin
open class User(val nickname: String)

class SocialUser(nickname: String) : User(nickname)
```

- base 클래스에 인자 넘기기도 가능. (약간 cpp 스러움)
- 당연히 User는 open 해야 함.

```kotlin
open class Button
class RadioButton : Button() // Button() 호출 필수
```

- 생성자를 따로 안 만들면, 인자 없는 생성자(default 생성자)가 자동 생성됨
- 아까 예제에서 인터페이스 구현 시 ()를 안붙여줬는데, 클래스 상속에서는 ()를 붙여야 하는 이유가 나옴. (base 클래스를 인스턴스화 해야하기에 …)

```kotlin
class Secretive private constructor(val name: String)
```

- `private constructor` 사용 → 외부에서 인스턴스화 못 함
- 나중에 동반 객체(companion object) 등에서 내부적으로 만들 수 있음
    - 자바에서는 유틸리티나 싱클톤 클래스에 대해서 명시할 방법이 애매해서 private 생성자를 정의하는 식으로 해소.
    - 킹틀린에서는 이런걸 언어 레벨에서 기본 제공해줌. (ex. object 등, 추후 설명 예정)

| 특징 | 설명 |
| --- | --- |
| 주 생성자 | 클래스명 옆 `()`에 정의 |
| `val`/`var` 파라미터 | 프로퍼티 자동 생성 |
| `init` 블록 | 초기화 코드 실행 장소 |
| 기본값 지원 | 파라미터 생략 가능 |
| 상속 시 호출 | 부모 생성자 호출 필수 |
| 비공개 생성자 | `private constructor` 사용 |

### 부 생성자 초기화 블록

**주 생성자 vs 부 생성자 차이**

| 구분 | 주 생성자 | 부 생성자 |
| --- | --- | --- |
| 정의 위치 | 클래스 선언 옆 `()` | 클래스 본문 안에서 `constructor` 키워드 사용 |
| 기본값 지원 | 가능 (`val x: Int = 0`) | 기본값 지원 X → 오버로딩으로만 처리 |
| 초기화 방식 | `init {}` 블록 사용 | 생성자 본문 안에서 직접 처리 |
| 상위 클래스 호출 | `: 부모(...)` | `super(...)` |
| 자기 생성자 호출 | 불가능 | `this(...)` 로 다른 생성자 호출 가능 |

```java
public class Downloader {
    public Downloader(String url) { ... }
    public Downloader(URI uri) { ... }
}
```

```kotlin
open class Downloader {
    constructor(url: String?) {
        // URL 초기화
    }

    constructor(uri: URI?) {
        // URI 초기화
    }
}
```

- `constructor` 키워드를 붙여서 부 생성자 선언
- 클래스 선언부에는 `()` 없음 → 주 생성자 없음!
    - 주 생성자() 없이 부 생성자만 선언 가능.

```kotlin
class MyDownloader : Downloader {
    constructor(url: String?) : super(url) {
        // 추가 초기화 코드
    }

    constructor(uri: URI?) : super(uri) {
        // 또 다른 초기화 코드
    }
}
```

- 상속할 때, 부 생성자 사용 가능.
- `super(...)`를 통해 상위 클래스 생성자 호출
- 생성자마다 다른 방식으로 초기화할 수 있도록 분리

```kotlin
class MyDownloader : Downloader {
    constructor(url: String?) : this(URI(url)) // 다른 생성자에 위임
    constructor(uri: URI?) : super(uri)
}

```

- `this(...)` → 같은 클래스의 다른 생성자 호출
- 문자열로 들어온 URL을 URI로 바꿔서 재사용하는 구조
- 주 생성자가 없을 경우, **모든 부 생성자는 반드시**
    - `super(...)`로 상위 클래스 초기화하거나,
    - `this(...)`로 자기 자신의 다른 생성자를 호출해야 함
- 결국 **생성자 호출 흐름이 끝나면 무조건 상위 클래스까지 초기화**되어야 함

**언제 부 생성자를 쓰면 좋은가?**

| 경우 | 이유 |
| --- | --- |
| 자바 상호운용 | 자바에선 기본값/이름 있는 인자 지원 안됨 → 생성자 여러 개 필요 |
| 다양한 초기화 방식 | URL/파일/경로 등 다양한 입력 형식 처리 필요 |
| 일부 인자 전처리 필요 | `String → URI`, `Map → JSON` 같은 변환 작업 후 초기화 |
- 기본적으로 코틀린은 **디폴트 파라미터 + 이름 지정 인자**로 충분
- 하지만 자바 호환성이나 복잡한 초기화 과정이 있으면 `constructor` 부 생성자 사용
- `super(...)` → 상위 클래스 생성자 호출
    
    `this(...)` → 같은 클래스의 다른 생성자 호출
    

## 인터페이스에 선언된 프로퍼티 구현

```kotlin
interface User {
    val nickname: String
}
```

- `nickname`은 추상 프로퍼티
- 즉, `User`를 구현하는 클래스는 반드시 `nickname`을 구현해야 함
- 인터페이스에는 상태(필드)가 없음 → 값을 저장할 수 없음 → 값을 저장하려면 **구현 클래스**에서 저장해야 함

```kotlin
// 1번
class PrivateUser(override val nickname: String) : User

// 2번
class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // 커스텀 게터
}

// 3번
class SocialUser(val accountId: Int) : User {
    override val nickname = getNameFromSocialNetwork(accountId)
}
```

1. 주 생성자에서 바로 프로퍼티 구현
    1. 실제로 nickname 이라는 프로퍼티가 생성
2. 커스텀 getter에서 계산해서 반환
    1. 값을 저장하지 않고 매 번 새로 계산함. → 계산 비용 trade-off 따져야 함.
3. 초기화할 때 계산된 값 저장
    1. 값을 딱 한 번 저장해서 재활용 → 저장 비용 trade-off 따져야 함.

```kotlin
interface EmailUser {
    val email: String
    val nickname: String
        get() = email.substringBefore('@') // 디폴트 구현
}
```

- 인터페이스에 커스텀 getter 를 사용하는 프로퍼티 포함 가능.
- `nickname`은 하위 클래스에서 override 안 해도 됨
- 대신 `email`은 추상 프로퍼티 → 반드시 구현해야 함

<aside>
💡

**함수 대신 프로퍼티를 써야 할 때**

아래 조건을 만족하면 val 프로퍼티 형태로 선언하는 것이 “코틀린” 스러운 것.

- 예외를 던지지 않음
- 계산 비용이 낮거나 **한 번만 계산해서 재사용** 가능
- 상태가 안 바뀌면 항상 같은 값을 반환함 (멱등성)
</aside>

## Getter와 Setter에서 뒷바침 필드 접근

### 뒷바침 필드?

> 프로퍼티 값을 저장해두는 진짜 저장소(= 변수)
> 
- 코틀린에서 프로퍼티를 선언하면,
    - 컴파일러가 내부적으로 `field`라는 숨겨진 변수를 만들어줌
- 이 `field`는 getter/setter 안에서만 사용할 수 있음
    
    → 밖에서는 접근 불가
    

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value) {
            println("Address was changed for $name: \"$field\" -> \"$value\"")
            field = value // 실제 값을 저장하는 부분
        }
}
```

```kotlin

val user = User("Alice")
user.address = "123 Kotlin Street"
// Address was changed for Alice: "unspecified" -> "123 Kotlin Street"
```

- `user.address = "123 Kotlin Street"`로 값을 바꾸면,
    - setter가 호출되고
    - 이전 값은 `field`로 가져오고
    - 새 값은 `field = value`로 저장함

```kotlin
class Person(var birthYear: Int) {
    var ageIn2050: Int
        get() = 2050 - birthYear
        set(value) {
            birthYear = 2050 - value
        }
}
```

- `ageIn2050`은 값을 저장하지 않음 → 단순 계산용
- `field`를 쓰지 않음 → 컴파일러는 **진짜 필드 안 만들어줌**

| 항목 | 내용 |
| --- | --- |
| `field` 키워드 | 프로퍼티 내부 getter/setter에서만 사용 가능 |
| getter에서는 | `field` 읽기만 가능 |
| setter에서는 | `field` 읽기 + 쓰기 가능 |
| 프로퍼티 값 저장 | `field = value` 로 꼭 저장해야 실제로 반영됨 |
| `field` 안 쓰면 | 계산용 프로퍼티 → 저장소 없이 동작 |

## 접근자 가시성 변경

```kotlin
var name: String = "hi"
```

- 기본적으로 `name`의 **게터와 세터는 둘 다 public**
- → `name = "new"` 도 외부에서 가능

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set // 외부에서는 값 설정 ❌, 내부에서는 가능 ✅

    fun addWord(word: String) {
        counter += word.length // 내부에서는 변경 가능
    }
}
```

- `private set/get` 을 통해 가시성 제어 가능

**사용 예**

```kotlin
fun main() {
    val lengthCounter = LengthCounter()

    lengthCounter.addWord("Hi!")
    println(lengthCounter.counter) // ✅ 읽기 OK → 3

    lengthCounter.counter = 0 // ❌ 오류! setter가 private이라 외부에서 수정 불가
}
```

- 값을 외부에 **노출은 하되, 변경은 막고 싶을 때** 유용함
    
    (예: 내부 계산 결과만 보여주고 외부 변경은 방지)
    
- Kotlin의 **캡슐화 기능을 섬세하게 조절**할 수 있음

# 컴파일러 생성 메소드 - 데이터 클래스 & 클래스 위임

## 모든 클래스가 정의해야 하는 메소드

1. `toString()`: 문자열로 객체를 표현하는 메서드

```kotlin
class Customer(val name: String, val postalCode: Int) {
    override fun toString() = "Customer(name=$name, postalCode=$postalCode)"
}
```

- 기본 `toString()`은 `Customer@e3f292b` 같은 형식이라 **별로 유용하지 않음**
- 오버라이드하면 디버깅이나 출력할 때 **가독성 높은 문자열 표현** 제공 가능

2. `equals()`: 객체의 **내용이 같은지** 비교하는 메서드

```kotlin
val c1 = Customer("Alice", 123)
val c2 = Customer("Alice", 123)

println(c1 == c2) // ❌ false → equals가 없으면 참조(주소) 비교
```

- 코틀린에서 `==` 는 `equals()`를 호출함
- 내용을 비교하려면 `equals()`를 직접 오버라이드해야 함

```kotlin
override fun equals(other: Any?): Boolean {
    if (other !is Customer) return false
    return name == other.name && postalCode == other.postalCode
}
```

3. `hashCode()`: 해시 기반 컬렉션(`HashSet`, `HashMap`)에서 필수

```kotlin
val set = hashSetOf(Customer("Alice", 123))
println(set.contains(Customer("Alice", 123))) // ❌ false

```

- 이유: `equals()`만 오버라이드하고 `hashCode()`는 안 고치면 **해시 컬렉션에서 잘못 작동함**
- **규칙**: `equals() == true`면 `hashCode()`도 반드시 같아야 함
    - 참고: https://cabi.oopy.io/a0b240ef-486b-4888-9b10-1f53219254cc

```kotlin
override fun hashCode(): Int = name.hashCode() * 31 + postalCode
```

***🤯 그런데 매번 이렇게 3개 메서드 다 오버라이드하는 건 귀찮고 번거로움***

## **해결책: `data class` 사용**

```kotlin
data class Customer(val name: String, val postalCode: Int)
```

- 킹틀린의 `data class`는
    - `toString()`
    - `equals()`
    - `hashCode()`
    를 **자동으로 생성해줌**
- 코드 한 줄로 완전한 동작 가능!

```kotlin
val c1 = Customer("Alice", 123)
val c2 = Customer("Alice", 123)

println(c1 == c2)            // ✅ true
println(c1.toString())       // ✅ Customer(name=Alice, postalCode=123)
println(setOf(c1).contains(c2)) // ✅ true
```

## 데이터 클래스와 불변성

왜 `val` 프로퍼티를 권장할까?

- 불변 객체는 **수정 불가능** → 더 안전하고 예측 가능
- 특히 다음 상황에서 **꼭 불변이어야 함**:
    - `HashMap`이나 `HashSet`의 키로 쓸 때
    - 멀티스레드 환경에서 공유할 때
- `var`로 만들면 나중에 값 바뀌어 버려서 문제가 생김 (예: 해시값 달라짐)

### `copy()` 메서드

- 데이터 클래스는 `copy()`라는 **자동 생성 메서드**를 제공함
- **전체를 복사하되 일부 값만 바꾸고 싶을 때 유용함**

```kotlin
val original = Customer("이계영", 4122)
val updated = original.copy(postalCode = 4800)

println(updated) // Customer(name=이계영, postalCode=4800)

```

- 원본은 그대로, 변경된 복사본을 생성 → **불변성 유지**
- 한마디로 `deepcopy`

### 자바의 레코드와 비교

| 항목 | Kotlin `data class` | Java `record` |
| --- | --- | --- |
| `copy()` 지원 | ✅ 있음 | ❌ 없음 |
| 상속 가능 | ✅ 가능 | ❌ 불가 |
| 필드 변경 | `val` or `var` | 모두 `final` |
| 추가 프로퍼티 정의 | 가능 | ❌ 불가능 |

## 클래스 위임 (`by` 키워드)

### 문제 상황

인터페이스 구현을 "내부 객체에게 넘기고 싶은데"

그걸 일일이 작성하면 너무 **복잡**함

```kotlin
class DelegatingCollection<T> : Collection<T> {
    private val innerList = ArrayList<T>()

    override fun size() = innerList.size
    override fun isEmpty() = innerList.isEmpty()
    ...
}
```

`by` 키워드로 위임하면?

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = ArrayList()
) : Collection<T> by innerList

```

- `Collection` 인터페이스 구현을 **자동으로 innerList에게 위임**
- 필요한 메서드만 선택적으로 `override` 가능

```kotlin
class CountingSet<T>(
    private val innerSet: MutableCollection<T> = HashSet()
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
```

```kotlin
fun main() {
    val cset = CountingSet<Int>()
    cset.addAll(listOf(1, 1, 2))
    println("Added ${cset.objectsAdded} objects, ${cset.size} uniques.")
    // 출력: Added 3 objects, 2 uniques.
}

```

- `add`와 `addAll`은 직접 구현해서 **카운팅 로직 추가**
- 나머지 메서드는 `by innerSet`으로 **자동 위임**

| 개념 | 요약 |
| --- | --- |
| `data class` | 불변 데이터 표현에 최적화, `toString`, `equals`, `copy()` 자동 생성 |
| `copy()` | 불변 객체의 일부 값만 바꾸고 복사본 만들기 |
| 불변성 | 멀티스레드 안정성, 해시 컨테이너의 정확성 보장 |
| 클래스 위임 (`by`) | 인터페이스 구현을 내부 객체에게 자동으로 위임 |
| 커스터마이징 | 필요한 메서드만 `override`해서 고유 로직 추가 가능 |

## 클래스, 객체, 인터페이스의 고오급 활용

### `object` 키워드의 3가지 용도

| 유형 | 설명 | 사용 예 |
| --- | --- | --- |
| **객체 선언 (`object Foo`)** | 싱글턴 객체 선언 | `object Payroll` |
| **동반 객체 (`companion object`)** | 클래스 내부에서 정적 멤버처럼 사용 | 팩토리 메서드 |
| **객체 식 (`object : ...`)** | 익명 객체 (자바의 익명 내부 클래스) | 이벤트 리스너 |

### 객체 선언 (`object`)

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()
    fun calculateSalary() { /* ... */ }
}
```

- **싱글턴 패턴**을 언어 차원에서 지원
- `Payroll.calculateSalary()`처럼 사용
- 생성자 없음, 객체는 **즉시 생성됨**

### 🔸 4.4.2 동반 객체 (`companion object`)

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
    }
}
```

- 클래스와 **함께 정의되는 싱글턴**
- 팩토리 메서드, 정적 멤버 대체
- 클래스 이름으로 접근 가능: `User.newSubscribingUser(...)`
- `private` 생성자와 조합하여 **팩토리 패턴** 구현에 적합

👉 **인터페이스 구현도 가능**:

```kotlin
interface Factory<T> {
    fun fromJSON(json: String): T
}

class Person(val name: String) {
    companion object : Factory<Person> {
        override fun fromJSON(json: String): Person { /*...*/ }
    }
}

fun <T> load(factory: Factory<T>) = factory.fromJSON("...")

load(Person) // 동반 객체가 인터페이스 역할

```

### 동반 객체 확장 함수

```kotlin
class Person(val name: String) {
    companion object
}

fun Person.Companion.fromJSON(json: String): Person { /* ... */ }

val p = Person.fromJSON("...") // 마치 클래스 메서드처럼 사용 가능

```

- 외부에서 정의하지만 마치 내부 정적 메서드처럼 사용할 수 있음
- 단, **빈 companion object라도 반드시 존재해야** 함

### 객체 식 (익명 객체)

```kotlin
Button(object : MouseListener {
    override fun onClick() { /*...*/ }
    override fun onEnter() { /*...*/ }
})

```

- 이름 없는 객체를 즉석에서 생성
- 자바의 **익명 내부 클래스**와 동일한 역할
- 함수 내부의 변수도 접근 가능 (`clickCount++` 등)

### 인라인 클래스 (inline class)

**왜 필요한가?**

함수 인자로 의미 있는 타입 구분을 하고 싶을 때

(예: 센트 vs 엔 단위, 이름 vs 아이디 등)

```kotlin
fun addExpense(expense: Int) // 문제: 단위가 불명확
```

### 인라인 클래스 선언

```kotlin
@JvmInline
value class UsdCent(val amount: Int)
```

- `val` 프로퍼티 **1개만** 가질 수 있음
- **JVM에는 감싸지 않고 인라인**되어 들어감 → 성능 손해 없음
- `class`는 만들지만 런타임엔 그냥 `Int`처럼 동작

| 항목 | 효과 |
| --- | --- |
| 타입 안전성 | 의미 없는 값 전달 방지 |
| 성능 | **객체 생성 없이** 컴파일러가 알아서 인라인 처리 |
| 유지보수 | 타입 이름이 의미를 명확히 해줌 |
- 인터페이스 구현 가능
- 계산된 프로퍼티 가능
- `copy()` 없음 (값 자체를 새로 만들어야 함)
- 상속 불가

```kotlin
interface Printable { fun prettyPrint() }

@JvmInline
value class UsdCent(val amount: Int) : Printable {
    val salesTax get() = amount * 0.06
    override fun prettyPrint() = println("$amount¢")
}

```

# 깜짝 퀴즈~

## 퀴즈 1. **객체 선언과 생성자**

```kotlin
object DatabaseManager {
    val url: String
    constructor(url: String) {
        this.url = url
    }
}

```

**Q. 위 코드는 컴파일될까? 된다면 그 이유는? 아니라면 그 이유는?**



## 퀴즈 2. **데이터 클래스, 불변성, HashSet**

```kotlin
data class Person(var name: String)

fun main() {
    val p1 = Person("Lee")
    val set = hashSetOf(p1)
    p1.name = "Kim"

    println(p1 in set) // ?
}

```

**Q. 위 코드는 무엇을 출력할까? 왜 그렇게 되는가?**


## 퀴즈 3. **클래스 위임과 오버라이딩 우선순위**

```kotlin
interface Greeter {
    fun greet(): String
}

class DefaultGreeter : Greeter {
    override fun greet() = "Hello from Default"
}

class CustomGreeter(g: Greeter) : Greeter by g {
    override fun greet() = "Hello from Custom"
}

fun main() {
    val greeter = CustomGreeter(DefaultGreeter())
    println(greeter.greet())
}

```

**Q. 출력 결과는? 그리고 왜 그런 결과가 나올까?**


