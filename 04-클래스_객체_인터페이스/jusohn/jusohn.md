# 04장 - (p.171 - p.236)

## 클래스와 인터페이스

- `sealed` 변경자는 클래스 상속이나 인터페이스 구현을 제한함

### 인터페이스

- 인터페이스 안에는 추상 메서드와 구현이 있는 메서드 모두 정의 가능
- 인터페이스에는 아무런 상태도 들어갈 수 없음

- 코틀린에서는 상속 (inheritance) 이나 구성 (composition) 에서 모두 클래스 이름 뒤에 콜론을 붙이고 인터페이스나 클래스 이름을 적음
- 클래스는 인터페이스를 원하는 만큼 구현할 수 있지만, 클래스는 오직 하나만 확장할 수 있음

- 상위 클래스나 상위 인터페이스에 있는 프로퍼티나 메서드를 오버라이드 할 때 `override` 변경자를 사용
- `override` 는 필수이며, 실수로 상위 클래스의 메서드를 오버라이드 하는 경우를 방지함

```kotlin
interface Clickable {
	fun click()
	fun showOff() = println("I'm clickable!")
}

interface Focusable {
	fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
	fun showOff() = println("I'm focusable!")
}

class Button: Clickable, Focusable {
	override fun click() = println("I was clicked")

	override fun showOff() {
		super<Clickable>.showOff()
		super<Focusable>.showOff()
	}
}

fun main(args: Array<String>) {
	val button = Button()
	button.showOFf()
	// I'm clickable!
	// I'm focusable!
	button.setFocus(true)
	// I got focus.
	button.click()
	// I was clicked.
}
```

- 같은 메서드를 구현하는 다른 인터페이스 정의가 있을 때, 한 클래스에서 이 두 인터페이스를 함께 구현하면 컴파일러 오류가 발생함
- 코틀린 컴파일러는 두 메서드를 아우르는 구현을 하위 클래스에 직접 구현하도록 강제함
  - 이름과 시그니처가 같은 멤버 메서드에 대해 둘 이상의 디폴트 구현이 있는 경우, 인터페이스를 구현하는 하위 클래스에서 명시적으로 새로운 구현을 제공해야 함

```java
class JavaButton implements Clickable {
	@Override
	public void click() {
		System.out.println("I was clicked");
	}
	@Override
	public void showOff() {
		System.out.println("I'm showing off");
	}
}
```

- 자바에서 코틀린의 인터페이스가 있는 메서드 구현하기
  - 코틀린은 디폴트 메서드가 있는 인터페이스를 일반 인터페이스와 디폴트 메서드 구현이 정적 메서드로 들어있는 클래스를 조합해 구현
    - 인터페이스에는 메서드 선언만 들어가며 인터페이스와 함께 생성되는 클래스에는 모든 디폴트 메사드 구현이 정적 메서드로 들어감
    - 따라서 디폴트 인터페이스가 포함된 코틀린 인터페이스를 자바 클래스에서 상속해 구현하고 싶다면 코틀린에서 메서드 본문을 제공하는 메서드를 포함하는 모든 메서드에 대한 본문을 작성해야 함

### open, final, abstract 변경자

- 코틀린에서 모든 클래스와 메서드는 기본적으로 final
- 어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 함
  - 더불어, 오버라이드를 허용하고 싶은 메서드나 프로퍼티의 앞에도 open 변경자를 붙여야 함
- 기반 클래스나 인터페이스의 멤버를 오버라이드 한 경우, 기본적으로 open 으로 간주됨
  - final 이 없는 override 메서드나 프로퍼티는 기본적으로 열려있음

```kotlin
abstract class Animated {
	abstract val animationSpeed: Double
		// 추상 프로퍼티. 하위 클래스는 반드시 값이나 접근자를 제공해야 함
	val keyframes: Int = 20
		// 추상 클래스의 (추상이 아닌) 프로퍼티는 기본적으로 열려있지 않음
	open val frames: Int = 60

	abstract fun animate()
		// 추상 함수. 하위 클래스는 이 함수를 반드시 오버라이드 해야 함
	open fun stopAnimating() { /* ... */ }
		// 추상 클래스의 (추상이 아닌) 함수는 기본적으로 열려있지 않음
	fun animateTwice() { /* ... */ }
```

- abstract 로 선언한 추상 클래스는 인스턴스화할 수 없음
  - 추상 멤버는 항상 열려 있음
  - 추상 클래스의 (추상이 아닌) 프로퍼티는 기본적으로 열려있지 않음
  - 추상 클래스의 (추상이 아닌) 함수는 기본적으로 열려있지 않음

![image.png](attachment:bb58e5af-8dd5-43fe-ac15-5bb1db087840:image.png)

### 가시성 변경자

- 코틀린에서 아무 변경자도 없는 선언은 모두 공개, 즉 public

![image.png](attachment:1175d2b3-6e8b-4ab5-93f1-78ca556fb9f4:image.png)

### 내부 클래스 (inner class) 와 내포 클래스 (nested class)

- 자바에서 내포 클래스를 static으로 선언하면 그 클래스를 둘러싼 바깥쪽 클래스에 대한 암시적인 참조가 사라짐
- 코틀린에서 내보된 클래스에 아무런 변경자도 없으면 자바 static 내포 클래스와 같음
  - 이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야 함

![image.png](attachment:f61fe1c6-5795-4954-9737-b10a4c18a3f7:image.png)

```kotlin
class Outer {
	inner class Inner {
		fun getOuterReference(): Outer = this@Outer
	}
}
```

- 내부 클래스 Inner 안에서 바깥쪽 클래스 Outer 의 참조에 접근하려면 this@Outer 라고 써야 함
- 클래스 계충을 만들되, 그 계층에 속한 클래스의 수를 제한하고 싶은 경우 내포 클래스를 쓰면 편리함

- 상위 클래스에 sealed 변경자를 붙이면, 그 상위 클래스를 상속한 하위 클래스의 가능성을 제한할 수 있음
  - sealed 클래스의 직접적인 하위 클래스들은 반드시 컴파일 시점에 알려져야 함
  - 봉인된 클래스가 정의된 패키지와 같은 패키지에 속해야 함
  - 모든 하위 클래스가 같은 모듈 안에 위치해야 함
  - sealed 변경자는 클래스가 추상 클래스임을 명시함 (abstract 를 붙일 필요가 없음)

## 생성자와 프로퍼티

- 어떤 클래스를 클래스 외부에서 인스턴스화하지 못하게 막고 싶다면, 생성자를 private 으로 만들어야 함

### 주 생성자 & 부 생성자

- 클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 함

### 인터페이스에 선언된 프로퍼티 구현

```kotlin
interface User {
	val nickname: String
}

class PrivateUser(override val nickname: String) : User

class SubstribingUser(val email: String) : User {
	override val nickname: String
		get() = email.substringBefore('@')
}

class SocialUser(val accountId: Int) : User {
	override val nickname = getFacebookName(accountId)
}

fun getNameFromSocialNetwork(accountId: Int) = "kodee$accountId"

fun main() {
	println(PrivateUser("kodee").nickname)
	// kodee
	println(SubscribingUser("test@kotlinlang.org").nickname)
	// test
	println(SocialUser(123).nickname)
	// kodee123
}
```

- 언제 함수 대신 프로퍼티를 사용할까?
  - 일반적으로
    - 클래스의 특성 → 프로퍼티
    - 클래스의 행동 → 메서드
  - 다음과 같은 특징을 만족하는 경우, 함수 대신 프로퍼티를 사용하라
    - 예외를 던지지 않음
    - 계산 비용이 적게 듬
    - 객체 상태가 바뀌지 않으면 여러 번 호출해도 항상 같은 결과를 돌려줌
  - 위에 해당하지 않으면, 함수를 사용하라

## 컴파일러가 생성한 메서드

## 데이터 클래스

## 클래스 위임

## object 키워드

## 요약

- 코틀린의 인터페이스는 자바 인터페이스와 비슷하지만 디폴트 구현과 프로퍼티도 포함할 수 있음
- 모든 코틀린 선언은 기본적으로 final이며 public 임
- 선언이 final 이 되지 않게 하려면 (상속과 오버라이딩이 가능하게 하려면), 앞에 open 을 붙여야 함
- internal 선언은 같은 모듈 안에서만 볼 수 있음
- 내포 클래스는 기본적으로 내부 클래스가 아님
  - 바깥쪽 클래스에 대한 참조를 내포 클래스 안에 포함시키려면 inner 키워드를 클래스 선언 앞에
    붙여야 함
- sealed 클래스를 직접 상속하는 클래스나, sealed 인터페이스에 대한 구현들은 모두 컴파일 시점에 컴파일러에 보여야 함
- 초기화 블록과 부 생성자를 활용해 클래스 인스턴스를 더 유연하게 초기화 할 수 있음
- field 식별자를 통해 프로퍼티 접근자 (게터와 세터) 안에서 프로퍼티의 데이터를 저장하는 데 뒷받침하는 필드를 참조할 수 있음
- 데이터 클래스를 사용하면 컴파일러가 equals, hashCode, toString, copy 등의 메서드를 자동으로 생성함
- 클래스 위임을 사용하면 위임 패턴을 구현할 때 비슷한 위임 코드를 반복하는 것을 피할 수 있음
- 객체 선언은 싱글턴 클래스를 정의하는 코틀린식 방법임
- (패키지 수준 함수와 프로퍼티와 더불어) 동반 객체는 자바의 정적 메서드와 필드 정의를 대신함
- 동반 객체도 다른 (싱글턴) 객체와 마찬가지로, 인터페이스를 구현할 수 있으며, 동반 객체에 대해서도 확장 함수와 프로퍼티를 정의할 수 있음
- 코틀린의 객체 식은 자바의 익명 내부 클래스를 대신하며, 여러 인터페이스를 구현하거나 객체가 포함된 영역에 있는 변수의 값을 변경하는 등, 자바 익명 내부 클래스보다 더 많은 기능을 제공함
- 인라인 클래스를 사용하면 수명이 짧은 객체를 많이 할당하므로써 발생할 수 있는 성능 저하를 피하면서도 프로그램에 새로운 타입 안정성 계층을 추가할 수 있음
