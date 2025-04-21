### 요약

- 코틀린의 인터페이스는 자바 인터페이스와 비슷하지만, 디폴트 구현과 프로퍼티도 포함할 수 있다.
- 모든 코틀린 선언은 기본적으로 final이며 public이다.
- 선언이 final이 되지 않게 만들려면(상속과 오버라이딩이 가능하게 하려면) 앞에 open을 붙여야 한다.
- internal 선언은 같은 모듈 안에서만 볼 수 있다.
- 내포 클래스는 기본적으로 내부 클래스가 아니다. 바깥쪽 클래스에 대한 참조를 내포 클래스 안에 포함시키려면 inner 키워드를 클래스 선언 앞에 붙여야 한다.
- sealed 클래스를 직접 상속하는 클래스나 sealed 인터페이스에 대한 구현들은 모두 컴파일 시점에 컴파일러에 보여야 한다.
- 초기화 블록과 부 생성자를 활용해 클래스 인스턴스를 더 유연하게 초기화할 수 있다.
- field 식별자를 통해 프로퍼티 접근자(게터와 세터) 안에서 프로퍼티의 데이터를 저장하는 데 뒷받침하는 필드를 참조할 수 있다.
- 데이터 클래스를 사용하면 컴파일러가 equals, hashCode, toString, copy 등의 메서드를 자동으로 생성해준다.
- 클래스 위임을 사용하면 위임 패턴을 구현할 때 비슷한 위임 코드를 반복하는 것을 피할 수 있다.
- 객체 선언은 싱글턴 클래스를 정의하는 코틀린식 방법이다.
- 패키지 수준 함수와 프로퍼티와 더불어, 동반 객체는 자바의 정적 메서드와 필드 정의를 대신한다.
- 동반 객체도 다른(싱글턴) 객체와 마찬가지로 인터페이스를 구현할 수 있으며, 동반 객체에 대해서도 확장 함수와 프로퍼티를 정의할 수 있다.
- 코틀린의 객체 식은 자바의 익명 내부 클래스를 대신하며, 여러 인터페이스를 구현하거나 객체가 포함된 영역에 있는 변수의 값을 변경할 수 있는 등 자바 익명 내부 클래스보다 더 많은 기능을 제공한다.
- 인라인 클래스를 사용하면 수명이 짧은 객체를 많이 할당함으로써 발생할 수 있는 성능 저하를 피하면서도, 프로그램에 새로운 타입 안전성 계층을 추가할 수 있다.

### 가장 중요한 것

각 class들 (data, open, sealed, inner…)과 interface, 그리고 object에 대한 이해. + data class와 copy()

- 내부 클래스(class)에서 암시적 외부 클래스에 대한 참조가 없다.
    
    → inner class로 explicit하게 해야함.
    
- 와 override가 겹치면 super를 generic으로 명시해서 사용할 수 있게 한다고? 대단하다.
- 기본이 final → 잘 열고 닫는게 중요하다.
- final이 없는 override method나 property는 open이다.
    - 가변이 가능한 경우(final이 아닌 경우) 스마트 캐스트를 사용할 수 없다.
        
        ```kotlin
        class FinalExample(val data: Any?) {
            fun printLength() {
                if (data is String) {
                    // data가 String으로 스마트 캐스트되어 별도의 명시적 캐스트 없이 length 접근 가능
                    println(data.length)
                }
            }
        }
        
        fun main() {
            val example = FinalExample("Hello")
            example.printLength() // 5 출력
        }
        
        //
        
        // open 프로퍼티로 선언된 경우
        open class OpenExample(open val data: Any?) {
            fun printLength() {
                if (data is String) {
                    // 컴파일 오류: "Smart cast to 'String' is impossible"
                    println(data.length)
                }
            }
        }
        
        class SubExample(override val data: Any?) : OpenExample(data)
        
        // 커스텀 접근자가 있는 경우
        class CustomGetterExample(val data: Any?) {
            // 커스텀 get()을 정의했으므로 스마트 캐스트 적용 불가
            val customData: Any?
                get() = data
        
            fun printCustomLength() {
                if (customData is String) {
                    // 컴파일 오류: 스마트 캐스트 불가
                    println(customData.length)
                }
            }
        }
        ```
        
        > `OpenExample`의 data는 `open val`로 선언되어 있고, 이를 상속한 `SubExample`에서 재정의될 수 있으므로 컴파일러는 스마트 캐스트를 보장하지 않습니다.
        > 
        
        > `CustomGetterExample`의 `customData`는 **커스텀 접근자**를 통해 값을 반환하므로 스마트 캐스트 대상이 아닙니다.
        > 
- abstract class의 경우도 아무 키워드가 없는 function이면 final이고, open하지 않으면 숨겨진다.
    
    → 그럼 abstract class 왜 씀? : 그것을 확장하게 될 자식 클래스가 반드시 수반해야 하는 기본적인 작동들이면서 함부로 변경할 수 없도록 하려고.. 가 내 생각
    

- internal 할거면 다 해놔라.
- internal은 코틀린 컴파일러가 애쓰는(mangling)부분. 귀엽네..
- inner class 용례
    - BottomSheetScope 자체에 Content들을 control하는 권한(기능)이 있지만, Content 자체에도 스스로를 종료할 수 있는 기능을 연결해주기(해당 content는 본인과 연결된 scope의 close만 사용가능함)
    
    ```kotlin
    abstract class OverlayScope<T : OverlayContent> {
        private val contents: SnapshotStateList<T> = mutableStateListOf()
    
        open fun open(content: T) {
            contents.add(content)
        }
        
        //...
    }
    
    interface OverlayContent {
        val id: OverlayId
        val dismiss: () -> Unit
        val composable: @Composable (OverlayId) -> Unit
    }
    
    class BottomSheetScope : OverlayScope<BottomSheetScope.Content>() {
    
        inner class Content(
            override val id: OverlayId = OverlayId(),
            val isBackgroundClickDismissable: Boolean = true,
            val isBackgroundDimmed: Boolean = true,
            val isDraggable: Boolean = true,
            val isHeightConstrained: Boolean = false,
            private val _isVisible: MutableState<Boolean> = mutableStateOf(true),
            val isVisible: State<Boolean> = _isVisible,
            override val dismiss: () -> Unit = {
                close(id) // <- 이 부분은 outer class의 function인데 해당 inner class에서 사용할 수 있도록 하기 위해서.
            },
            override val composable: @Composable (OverlayId) -> Unit
        ) : OverlayContent {
            fun hide() {
                _isVisible.value = false
            }
        }
    }
    ```
    
- sealed의 주요 용처 (안드로이드의 경우)
    
    ```kotlin
    sealed interface LauncherState {
        object Loading : LauncherState
        data class UpdateRequired(val updateRequirement: UpdateRequirement) : LauncherState
        data class Error(val message: String?) : LauncherState
    }
    
    //.. when을 이용한 표현
    
    when (state) {
        is LauncherState.Loading -> // 로딩 뷰
        is LauncherState.UpdateRequired -> // 업데이트 뷰
        is LauncherState.Error -> // 에러 뷰
    }
    ```
    
- init과 kotlin의 require를 잘 이용해보자.
    
    ```kotlin
    @JvmInline
    value class HourMinute(val value: String = "00_00") {
    	init {
    	    val parts = value.split("_")
    	    require(parts.size == 2) { "Invalid HourMinute format: $value" }
    	    val hour = parts[0].toIntOrNull()
    	    val minute = parts[1].toIntOrNull()
    	    require(hour in 0..24) { "Invalid hour: ${parts[0]}" }
    	    require(minute in 0..59) { "Invalid minute: ${parts[1]}" }
    	    if (hour == 24) require(minute == 0)
    	}
    }
    ```
    
- toString, equals, hashCode등은 아는 내용이라 스킵
- data class & copy는 진짜 밥먹듯이 쓰게되는 부분
    
    예시는 상태가 변경되는 도메인 객체에 대한 부분이다.
    
    ```kotlin
    fun copyAsUpdated(now: LocalDateTime = LocalDateTime.now()): NormalSession {
        if (!shouldBeUpdated(now)) return this
        return copy().apply {
            when (state) {
                PolicyState.IDLE, PolicyState.DEACTIVATED, PolicyState.PAUSED -> activate()
                PolicyState.ACTIVE -> transitionToIdle()
            }
        }
    }
    ```
    
- 책에서 상속 대신 언급하는 데코레이터 패턴은 이전 클래스의 interface를 구현하되 내부에 해당 구현체를 담는다는 부분에서 조금 더 엄격한 컴포지션 패턴처럼 느껴진다.
    - delegation을 이용해서 collection을 잘 사용한다면, [일급 컬렉션](https://jojoldu.tistory.com/412)을 잘 사용하기 좋을 것 같다.
- `object`는 고정된 인스턴스다 → 당연히 테스트가 어려울 것
    - 단순한 유틸 클래스로서 사용하는 것이 좋지 않을까? (stateless)
    - 책에 바로 나온다.

      <img width="619" alt="스크린샷 2025-04-21 오전 9 26 22" src="https://github.com/user-attachments/assets/51c2859d-194d-45ec-badd-344c50913030" />

        
- static 생성자와 `companion object`
    - named companion object로 Builder도 편하게 만들 수 있을 것
    - 해당 클래스의 전체에 해당하는 interface를 companion object를 통해 구현할 수 있다.
- 익명객체 - object : Interface<?> { … }
    - 실제로 특정 라이브러리에서 단순히 한 번 사용하고 마는 다이나믹한 객체들의 경우 이런 식으로 많이 사용
- inline class ← VO를 만들기에 좋다.

문제

1. UI를 나타내는 다음과 같은 단계가 있다.
    - 로딩
    - 로드됨(title: String, description: String)
    - 에러(message : String?)
    
    위 단계를 State를 sealed interface를 이용해서 구현해보자.
    

[1번 답]()

1. 계산기를 구현해보자. 요구사항은 다음과 같다.
    - 계산기는 `data class` 다.
    - 현재 값을 기억하고 있다.
    - 덧셈만 제공한다.
    - `thread-safe` 해야 한다.
    - 최대한 간단하게 구현한다 - edge case는 고려하지 않는다

[2번 답]()

1. 위 계산기에서 쓰였던 amount를 항상 양수임을 보장하는 value class로 재구성해보자.

[3번 답]()
