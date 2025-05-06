# 8. 기본 타입, 컬렉션, 배열

## 8.1 원시 타입과 기본 타입
- 자바와 달리 코틀린은 원시 타입과 래퍼 타입을 구분하지 않음.
  - 항상 같은 타입 사용. -> 더 편리, 원시 타입의 값에 대해 메서드 호출 가능
- 컴파일 시 대부분의 경우 원시 타입으로 컴파일, 불가능한 경우(ex. 제네릭 클래스 사용)에만 래퍼타입으로 컴파일

### 부호 없는 숫자 타입
- UByte, UShort, UInt, ULong
- 명시적으로 전체 비트 범위가 필요한 경우가 아니라면 일반적으로 보통의 정수를 사용하고 음수 범위가 함수에 전달됐는지 검사하는 편이 나음.
- 내부에서 UInt 값은 실제로 Int(인라인 처리로 가공)

### 널이 될 수 있는 기본 타입(Int?, Boolean? 등)
- 널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 표현할 수 없음.
- 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일
- 널이 될 가능성이 있으면 두 값을 직접 비교 x, 먼저 두 값이 모두 널이 아닌지 검사
- 제네릭 클래스의 경우 래퍼 타입 사용. 어떤 클래스의 타입 인자로 원시 타입을 넘기면 코틀린은 그 타입에 대한 박스 타입 사용
  - ex. `val listOfInts = listOf(1, 2, 3)`
  - JVM은 타입 인자로 원시 타입을 허용하지 않기 때문

### 수 변환
- 코틀린은 한 타입의 수를 다른 타입의 수로 자동변환하지 않음.
- 모든 원시 타입(Boolean 제외)에 대해 toByte(), toShort(), toChar() 등의 변환 함수 제공
  - `Int.toLong()`, `Long.toInt()`...
```kotlin
val x = 1
val list = listOf(1L, 2L, 3L)
x in list // false(다른 타입끼리 비교)
```
- 코틀린에서는 타입을 명시적으로 변환해서 같은 타입의 값으로 만든 후 비교해야 함.
> **원시 타입 리터럴** <br/>
> 코틀린은 소스코드에서 단순한 10진수(정수) 외에 수 리터럴 허용
> - Long 타입: 123L
> - Double 타입: 0.12, 2.0, 1.2e10, 1.2e-10
> - Flout 타입: 123.4f, .456F, 1e3f
> - 16진 리터럴: 0xCAFEBABE, 0XbcdL
> - 2진 리터럴: 0b000000101, 0B000000101 <br/>
> 
> 문자 리터럴의 경우 작은따옴표 안에 문자
>   - '1', '\t', '\u009'(유니코드)...

- 직접 변환하지 않더라도 숫자 리터럴을 타입이 알려진 변수에 대입하거나 함수에게 인자로 넘기면 컴파일러가 필요한 변환을 자동으로 넣어줌.

> [!NOTE]
> **문자열을 수로 변환하기**
> - 코틀린 표준 라이브러리는 문자열을 원시 타입으로 변환하는 여러 함수 제공
>   - 파싱에 실패하면 NumberFormatException 발생
> - 각 확장함수에는 변환에 실패하면 null을 돌려주는 함수 존재(toIntOrNull, toByteOrNull...)
> - toBoolean 함수는 인자로 받은 문자열이 null이 아니고 단어가 true이면 true 반환
>   - 정확히 true, false와 일치시키려면 toBooleanStrict 함수 사용

### Any, Any?: 코틀린 타입 계층의 뿌리
- 코틀린에서는 Any타입이 모든 널이 될 수 없는 타입의 조상 타입
- 원시 타입 값을 Any타입의 변수에 대입하면 자동으로 값을 객체로 감쌈
- Any타입은 java.lang.Object에 대응
- 모든 코틀린 클래스에는 toString, equals, hashCode 메서드 존재. Any에 정의된 메서드를 상속한 것

### Unit 타입: 코틀린의 void
- 코틀린 함수의 반환 타입이 Unit이고 그 함수가 제네릭 함수를 오버라이드하지 않는다면 그 함수는 내부에서 자바 void 함수로 컴파일
- Unit은 타입 인자로 쓸 수 있음.(자바의 void는 불가)
- Unit에는 Unit이라는 값 하나 가지고있음. 암시적으로 반환
- 제네릭 파라미터를 반환하는 함수를 오버라이드하면서 반환 타입으로 Unit을 쓸 때 쓸모있음.
  ```kotlin
  interface Processor<T> {
    fun process(): T
  }

  class NoResultProcessor : Processor<Unit> {
    override fun process() { // Unit을 반환하지만 타입 지정 필요 없음.
      // 업무 처리 코드
    } // 명시적으로 return할 필요 없음.컴파일러가 암시적으로 return Unit
  }
  ```

### Nothing 타입: 이 함수는 결코 반환되지 않는다.
- 결코 성공적으로 값을 돌려주는 일이 없으므로 '반환값'이라는 개념 자체가 의미없는 함수가 일부 존재
  - fail(무조건 예외 던지는 함수), 무한 루프를 도는 함수...
- 그런 경우를 표현하기 위함.
- Nothing을 반환하는 함수를 엘비스 연산자의 오른쪽에 사용해서 전제조건 검사 가능
  ```kotlin
  val address = company.address ?: fail("No address")
  priontln(address.city)
  ```
- 컴파일러는 `company.address`가 null인 경우 엘비스 연산자의 오른쪽에서 예외가 발생한다는 사실 파악. address의 값이 null이 아님을 추론.

## 8.2 컬렉션과 배열

### '널이 될 수 있는 값'의 컬렉션과 '널이 될 수 있는 컬렉션'
- 컬렉션에 null을 넣을 수 있는지 여부는 어떤 변수의 값이 null이 될 수 있는지 여부와 마찬가지로 중요
- 널이 될 수 있는 Int로 이뤄진 리스트와 Int로 이뤄진 널이 될 수 있는 리스트 사이의 차이 <br/>
  ![image](https://github.com/user-attachments/assets/c1ec592d-8b12-48ba-95b6-4cbab9d5d797)
- 널이 될 수 있는 값으로 이뤄진 널이 될 수 있는 리스트 정의 -> `List<Int?>?`
- 이런 리스트를 처리할 때는 변수에 대해 null 검사 수행 후 그 리스트에 속한 모든 원소에 대해 다시 null 검사 수행 필요
- 널이 될 수 있는 값으로 이뤄진 컬렉션으로 null 값을 걸러내는 경우가 자주 있어서 filterNotNull이라는 함수 제공(mapNotNull도 있음)

### 읽기 전용과 변경 가능한 컬렉션
- 코틀린에서는 컬렉션 안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리함.
- `kotlin.collections.Collection` <-> `kotlin.collections.MutableCollection'
- MutableCollection은 Collection을 확장하면서 원소를 추가하거나, 삭제하거나, clear하는 등의 메서드 좀 더 제공. <br/>
![image](https://github.com/user-attachments/assets/e152a342-9b1c-4270-82c6-a4b73bfb119d)
- 가능하면 코드에서 항상 읽기 전용 인터페이스를 사용하는 것을 일반적인 규칙으로 삼자
- 어떤 컴포넌트의 내부 상태에 컬렉션이 포함된다면 그 컬렉션을 MutableCollection을 인자로 받는 함수에 전달할 때는 원본의 변경을 막고자 
  컬렉션을 복사해야 할 수도 있음(방어적 복사)
- 객체가 실제로는 변경가능한 컬렉션이어도 읽기 전용 컬렉션 타입이면 함수 인자로 넘길 수 없음.
- **읽기 전용 컬렉션이더라도 꼭 변경 불가능한 컬렉션일 필요는 없음.**
- 읽기 전용 인터페이스 타입인 변수를 사용할 때 그 인터페이스는 실제로는 어떤 컬렉션 인스턴스를 가리키는 수많은 참조 중 하나일 수 있음.<br/>
![image](https://github.com/user-attachments/assets/7b435dae-d764-48bf-8033-25d656978e17)
- 코드의 일부분이 가변 컬렉션에 대한 참조를 갖고 있고 다른 부분에서 같은 컬렉션에 대한 '뷰'를 갖고 있다면 후자의 코드는 전자가 컬렉션을
  동시에 변경할 수 없다는 가정에 의존할 수 없음.
- 읽기 전용 컬렉션이 항상 thread safe 하지 않음.
- 다중 스레드 환경에서 데이터를 다루는 경우 그 데이터를 적절히 동기화하거나 동시 접근을 허용하는 데이터 구조를 활용해야 함.

### 코틀린 컬렉션과 자바 컬렉션은 밀접히 연관됨
![image](https://github.com/user-attachments/assets/d590a996-e913-4d68-80a7-2392bb428e57)
- 코틀린의 읽기 전용과 변경 가능 인터페이스의 기본 구조는 java.util 패키지에 있는 자바 컬렉션 인터페이스의 구조와 같음.
- 각 변경 가능 인터페이스는 자신과 대응하는 읽기 전용 인터페이스 확장(상속)
- 읽기 전용 인터페이스에는 컬렉션을 변경할 수 있는 모든 요소가 빠져있음.
- Map 클래스도 Map과 MutableMap 2가지 버전
- setOf(), mapOf()는 Set과 Map 읽기 전용 인터페이스의 인스턴스를 반환하지만 내부적으로는 변경 가능한 클래스.
  - setOf, MapOf의 반환 값이 변경 가능한 클래스라는 사실에 의존x, 나중에 불변 컬렉션 인스턴스를 반환하게 바뀔 수도 있음.
- 코틀린에서 읽기 전용 Collection으로 선언된 객체라도 자바 코드에서는 내용 변경 가능
- 컬렉션을 변경하는 자바 메서드에게 읽기 전용 Collection을 넘겨도 코틀린 컴파일러가 이를 막을 수 없음.
- 널이 아닌 원소로 이뤄진 컬렉션을 자바 메서드로 넘겼는데, 자바 메서드가 null을 컬렉션에 넣을 수도 있음.
- 컬렉션은 자바 코드에게 넘길 때는 특별히 주의.

### 자바에서 선언한 컬렉션은 코틀린에서 플랫폼 타입으로 보임
- 컬렉션 타입이 파라미터로 들어간 자바 메서드 구현을 오버라이드 하는 경우 어떤 컬렉션 타입으로 표현할지 결정해야 함.
  - 컬렉션이 null이 될 수 있는가?
  - 컬렉션의 원소가 null이 될 수 있는가?
  - 여러분이 작성할 메서드가 컬렉션을 변경할 수 있는가?

### 성능, 상호운용을 위해 객체의 배열이나 원시 타입의 배열을 만들기
- 기본적으로는 배열보다는 컬렉션을 더 먼저 사용해야 함. but, 여러 자바 API가 여전히 배열을 사용하므로 배열을 써야하는 경우가 생김.
  - ex. 배열을 인자로 받는 자바 함수 호출 or vararg파라미터를 받는 코틀린 함수 호출
- 코틀린에서 배열을 만드는 방법
  - arrayof 함수는 인자로 받은 원소들을 포함하는 배열을 만듬
  - arrayOfNulls함수는 모든 원소가 null인 정해진 크기의 배열을 만들 수 있음.
  - Array 생성자는 배열 크기와 람다를 인자로 받아 람다를 호출해서 각 배열 원소를 초기화해줌.
- 람다는 배열 원소의 인덱스를 인자로 받아 배열의 해당 위치에 들어갈 원소 반환.
- `toTypedArray`: 컬렉션을 배열로 바꿔 줌.
  ```kotlin
  fun main() {
    val strings = listOf("a", "b", "c")
    println("%s/%s/%s".format(*strings.toTypedArray())) // vararg 인자를 넘기기 위해 스프레드 연산자 사용
  ```
- 배열 타입의 타입 인자도 항상 객체 타입(박싱)
- 박싱하지 않는 **원시 타입의 배열**이 필요하다면 특별한 배열 클래스 사용(IntArray, ByteArray, CharArray...)
- **원시 타입의 배열을 만드는 방법**
  - 각 배열 타입의 생성자는 size 인자를 받아 해당 원시 타입의 기본값(보통 0)으로 초기화된 size 크기의 배열 반환
  - 팩토리 함수(ex. intArrayOf)는 여러 값을 가변 인자로 받아 그런 값이 들어간 배열 반환
  - 크기와 람다를 인자로 받는 다른 생성자 사용
- 박싱된 값이 들어있는 컬렉션이나 배열이 있다면 toIntArray 등의 변환 함수를 사용해 원시 타입 값이 들어있는 배열로 변환 가능
- 코틀린 표준 라이브러리는 배열 기본 연산 + 컬렉션에 사용할 수 있는 모든 확장 함수(반환값은 배열이 아니라 리스트)를 배열에도 제공함.
  ```kotlin
  fun main(args: Array<String>) {
    args.forEachIndexed { index, element ->
      println("Argument $index is: $element")
    }
  }
  ```

## 요약
- 기본적인 수를 표현하는 타입은 일반 클래스처럼 보이고 동작하지만 보통 자바의 원시 타입으로 컴파일.
- 널이 될 수 있는 원시 타입은 박싱
- Any타입은 모든 다른 타입의 상위 타입. 자바의 Object 타입
- 자바에서 온 타입은 코틀린에서 플랫폼 타입으로 취급
- 코틀린은 읽기 전용과 변경 가능한 컬렉션을 구분함으로써 컬렉션 더 개선
- 코틀린에서 자바 클래스를 확장하거나 인터페이스를 구현해야 한다면 파라미터의 널 가능성과 변경 가능성을 주의 깊게 생각해야 함.
- 코틀린에서도 배열 사용가능. but, 일반적으로 컬렉션 사용하는 쪽을 권장
- 코틀린 Array는 제네릭클래스처럼 보이지만 자바 배열로 컴파일
- 원시 타입의 배열은 IntArray 등의 특별한 클래스로 표현

### Q1. nullable Int 배열을 받아서 null이 아닌 값들만 제곱하여 List<Int>로 반환하는 함수를 작성하시오
```kotlin
fun squareNotNull(input: Array<Int?>): List<Int> {

}
```

### Q2.
1) IntArray와 Array<Int>의 차이를 설명하시오. <br/><br/>
2) 정수를 vararg로 받아 짝수만 모아 IntArray로 반환하는 함수 작성하세요
   ```kotlin
   fun evensToIntArray(vararg numbers: Int): IntArray
   ```
   <br/><br/>
3) 정수를 vararg로 받아 짝수만 모아 Array<Int>로 반환하는 함수 작성하세요
   ```kotlin
   fun oddsToArray(vararg numbers: Int): Array<Int>
   ```
   <br/><br/>
4) IntArray 변수가 있을 때 vararg로 함수에 넘기려면 어떻게 해야하나요?<br/><br/>


### A1.
```kotlin
fun squareNotNull(input: Array<Int?>): List<Int> {
    input.filterNotNull()
         .map { it * it }
}
```
<br/><br/>
### A2.
1) IntArray는 정수형 원시타입으로 저장, Array<Int>는 래핑하여 객체타입으로 저장. IntArray가 성능상 유리 / Array<Int>는 숫자 특화 함수 사용x(sum(), average(), maxOrNull()등의 함수를 직접 사용 불가)
<br/><br/>
3)
```kotlin
fun evensToIntArray(vararg numbers: Int) =
    numbers.filter { it % 2 == 0 }
        .toIntArray()
```
<br/><br/>
3)
```kotlin
fun oddsToArray(vararg numbers: Int) =
    numbers.filter { it % 2 != 0 }
        .toTypedArray()
```
<br/><br/>
4) 스프레드 연산자(*) 사용하면 됩니다!



