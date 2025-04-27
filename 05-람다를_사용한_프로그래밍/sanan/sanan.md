- 요약
    - 람다를 사용하면 코드 조각을 다른 함수에게 인자로 넘길 수 있고, 따라서 공통 코드 구조를 쉽게 추출할 수 있다.
    - 코틀린에서는 람다가 함수 인자인 경우 괄호 밖으로 람다를 빼내어 코드를 더 깔끔하고 간결하게 만들 수 있다.
    - 람다의 인자가 단 하나뿐인 경우 인자 이름을 지정하지 않고 it이라는 디폴트 이름으로 부를 수 있다. 이를 통해 짧고 간단한 람다 내부에서 파라미터 이름을 붙이느라 노력할 필요가 없어진다.
    - 람다는 외부 변수를 캡처할 수 있다. 이는 예를 들어, 람다 안에 있는 코드가 람다 정의가 들어있는 바깥 함수의 변수를 읽거나 쓸 수 있다는 뜻이다.
    - 메서드, 생성자, 프로퍼티의 이름 앞에 ::을 붙이면 각각에 대한 참조를 만들 수 있다. 이런 참조를 람다 대신 다른 함수에 넘길 수 있다.
    - filter, map, all, any 등의 함수를 활용하면 직접 원소를 이터레이션하지 않고도 컬렉션에 대한 대부분의 연산을 수행할 수 있다.
    - 추상 메서드가 단 하나뿐인 인터페이스(SAM - Single Abstract method - 인터페이스라고도 알려짐)를 구현하기 위해 그 인터페이스를 구현하는 객체를 생성하는 대신 람다를 직접 넘길 수 있다.
    - 수신 객체 지정 람다를 사용하면 람다 안에서 미리 정해둔 수신 객체의 메서드를 직접 호출할 수 있다. 이 람다의 본문은 그 본문을 둘러싼 코드와는 다른 콘텍스트에서 작동하기 때문에 코드를 구조화할 때 도움이 된다.
    - 표준 라이브러리의 with 함수를 사용하면 어떤 객체에 대한 참조를 반복해서 언급하지 않으면서 그 객체의 메서드를 호출할 수 있다.
    - apply를 사용하면 어떤 객체이든 빌더 스타일의 API를 통해 생성하고 초기화할 수 있다.
    - also를 사용하면 객체에 대해 추가 작업을 수행할 수 있다.
- run { }를 이용하면..
    
    ```kotlin
    val notNullResult = some.resultMayBeNull()?: run {
        onResultNull()
        return
    }
    ```
    
    과 같이 null-safety하게도 사용할 수 있음.
    
- Lambda를 이용한 mapping
    
    ```kotlin
    // Int를 이용한 값으로 표현되는 iterable에 대한 picker
    // unit을 "초" 라고 인자를 전달하고
    // mapper를 만약 mapper = { ">$it<"} 과 같이 전달한다면
    // >1< 초 / >2< 초 와 같이 피커가 구성되는 것이다.
    @Composable
    fun UnitPicker(
    		items: Iterable<Int>,
        startIndex: Int,
        unit: String,
        modifier: Modifier = Modifier,
        mapper: (Int) -> String = { it.toString() },
        onItemSelected: (Int) -> Unit,
    ) {
        Row(
            modifier = modifier,
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            TDSPicker(
                modifier = Modifier,
                items = items,
                mapper = mapper,
                startIndex = startIndex,
            ) { selected ->
                onItemSelected(selected)
            }
            Text(text = unit.styleText(TDSFonts.BodyXL), color = TDSColors.Gray._50)
        }
    }
    ```
    
- 항상 컴파일러가 타입 추론을 잘 하는 것을 고려하고, 더해서 우리 스스로도 추론하자. + (읽기만해도 타입 추론이 자연스럽게 코드를 작성하는게 좋은 코드 아닐까? - 하지만 반대로 상세하게 설명하고 싶은 부분이나, 추상화를 많이 이용하는 영역에서는 의도적으로 길게 쓰는 편이다)
- 한편, 람중람(람다 안에 람다)과 같은 중첩 구조가 될 때에는 it 키워드를 마구 사용하지 말고, 적절한 변수명을 지정하여 가독성을 향상시키자. 이건 안 하면 협업 못 하는 프로그래머 같은 느낌.
    
    ```kotlin
    "example".let {
        "$it <- example"
    }.let { 
        it // it == "example <- example"
    }
    ```
    
- 코틀린은 람다 안에서 변수 바꾸는 거 쉽게 해줌.
    - 외부 변수를 캡처해서 쓰는데, 기존 Java에서는 final만 가능했음
    - 하지만 kotlin에서는 이 부분도 알아서 ref등의 캡처로 변환해서 수행하므로 편하게 쓸 수 있음.
    - 비동기에서는 변수 바꾸는 게 바로 보이지 않을 수 있음.
    - 좀 더 깊게
        
        ✅ **코틀린에서 람다를 쓰면, 컴파일할 때**
        
        → **컴파일러가 알아서** 그 람다를 **“익명 클래스”나 “함수형 인터페이스”로 변환**
        
        ✅ 만약 **람다 안에서 바깥 변수를 캡처**했다면
        
        → 그 변수도 **래핑(박스처럼 감싸서)**.
        
        ✅ 이 래핑하는 과정은 **컴파일 타임(코드를 컴파일할 때)에 발생**.
        
        ✅ 즉, **우리가 따로 신경 쓸 필요 없이** 컴파일러가 자동으로
        
        - 변수 복사할지
        - 래핑할지
        - 클래스 만들지
            
            결정하고, **최적화도 진행**.
            
        
        ### **1. 코틀린 람다가 만드는 오버헤드란 무엇인가?**
        
        ✅ 코틀린에서 **람다를 쓰면**, 컴파일러가 **익명 클래스(Anonymous Class)를 생성**.
        
        ✅ 이 과정에서 생기는 오버헤드는 크게 두 가지:
        
        - **힙 메모리 할당(heap allocation)**
        - **가비지 컬렉션 부하(GC pressure)**
        
        **왜냐하면:**
        
        - 람다가 객체처럼 메모리에 올라가고
        - 필요 없으면 GC가 치워야 하니까
        - → 그래서 “람다 하나 = 힙에 객체 하나”로 보면 됨.
        
        ---
        
        ### **2. 람다가 변수 캡처할 때 오버헤드는 더 커진다**
        
        ✅ 만약 **람다 안에서 바깥 변수를 캡처**했다면?
        
        - 캡처한 변수를 **래핑(wrapping)** 해야 함.
        - 이 래핑 자체도 **추가 객체**를 만듦.
        
        **즉,**
        
        - 람다 자체 객체 + 캡처한 변수용 래퍼 객체
        - → **메모리 할당이 두 번** 일어나는 셈.
        
        특히 캡처하는 변수가 많아질수록
        
        - 객체 크기 커지고
        - 참조 관리 복잡해지고
        - GC 비용도 높아짐.
        
        한편, inline 키워드를 붙이면 아예 컴파일타임에 해당 코드블럭을 붙여버리기 때문에 overhead가 줄어들 수 있음(그렇지만 대부분은 inline으로 작성해도 그렇게 크게 효율이 달라지거나 하지는 않음)
        

```kotlin
fun runTasks(tasks: List<() -> Unit>) {
    tasks.forEach { task ->
        Thread {
            task()
        }.start()
    }
}
// 619, 678, 648

inline fun runTasks2(tasks: List<() -> Unit>) {
    tasks.forEach { task ->
        Thread {
            task()
        }.start()
    }
}

// 631, 627, 656

fun main() {
    val responses = List(1000) { index ->
        { println("Running task #$index") }
    }
    runTasks(responses)
}
```

- with
    
    ```kotlin
    data class Person(var name: String)
    
    fun main() {
        val sanan = Person("Sanan")
        
        with(sanan) {
    		    // 해당 scope에서 해당 변수를 수신객체 this로 참조
            name = "Sanan <- blackholed"
        }
    }
    ```
    

문제 1

name: String을 갖는 Person이라는 data class가 있다고 하자.

이 클래스에서 이름을 수신 객체로 전달하고, Unit을 반환하는 람다를 매개변수로 갖는 함수를 작성해라.

```kotlin
data class Person(var name: String) {
    fun doSomethingWithName(lambda: (String) -> Unit) {
        lambda(name)
    }
}

val sanan = Person("Sanan")

sanan.doSomethingWithName {
    print("Hi my name is $it")
}
```

문제 2

문자열이 담긴 Iterable에 대해서, 가장 긴 순서대로 내림차순으로 정렬한 배열을 다음과 같은 방법으로 2가지 만들어라.

- sortedByDescending을 이용한다.
- 멤버 참조를 이용한다
- 람다식을 이용한다

```kotlin
val fruits = listOf("사과", "바나나", "아 샤인머스켓 먹고싶다")
val sorted = fruits.sortedByDescending(String::length)
val sorted2 = fruits.sortedByDescending { it.length }
```
