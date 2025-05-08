- 요약
    - 기본적인 수를 표현하는 타입(예: Int)은 일반 클래스처럼 보이고 동작하지만, 보통 자바의 **원시 타입(primitive type)**으로 컴파일된다.
    - 코틀린의 **부호 없는 수 클래스**(UInt, ULong 등)는 JVM에 상응하는 타입이 없지만, **인라인 클래스**로 변환되어 **원시 타입과 유사한 성능**을 낸다.
    - **널이 될 수 있는 원시 타입**(예: Int?)은 자바의 **박싱된 원시 타입**(예: java.lang.Integer)에 대응된다.
    - Any 타입은 **모든 다른 타입의 상위 타입**이며, 자바의 Object 타입에 대응된다.
    - Unit 타입은 자바의 void에 대응된다.
    - Nothing 타입은 **함수가 정상적으로 끝나지 않음**을 나타내는 타입이다. 예: 예외를 던지는 함수.
    - 자바에서 온 타입은 코틀린에서 **플랫폼 타입(platform type)**으로 취급된다.
        - 개발자는 이를 **널이 될 수 있는 타입**으로도, **널이 될 수 없는 타입**으로도 취급할 수 있다.
    - 코틀린은 컬렉션에 대해 **표준 자바 클래스**를 사용하지만, **읽기 전용과 변경 가능한 컬렉션을 구분**함으로써 컬렉션을 보다 개선했다.
    - 코틀린에서 자바 클래스를 **확장하거나 인터페이스를 구현**할 경우, **파라미터의 널 가능성과 변경 가능성**을 주의 깊게 고려해야 한다.
    - 코틀린에서도 배열을 사용할 수 있지만, 일반적으로는 **컬렉션 사용을 권장**한다.
    - 코틀린의 Array는 일반적인 **제네릭 클래스처럼 보이지만**, **자바 배열**로 컴파일된다.
    - **원시 타입의 배열**은 IntArray, DoubleArray 등의 **특별한 클래스**로 표현된다.
- 코틀린에서 제공하는 원시 타입에 해당할 수 있는 값들은 알아서 래퍼/원시 타입으로 런타임에서 동작한다(컴파일 타임에 잘 정돈된다는 얘기로 이해) - 일반 사용시 원시, 제네릭 등에 사용되는 경우 래퍼타입.
- 원시타입은 nullable하지 않으므로(레퍼런스의 메모리를 가리키는 것이 아닌 값을 담고 있으므로), nullable한 값 타입으로 지정하거나 추론시키는 경우 래퍼 타입으로 간주된다.
- 제네릭의 타입 인자는 래퍼 타입으로 사용된다. 필요하다면 `IntArray` 등을 이용할 수 있다.
- `Any`는 Not-Null이다. 원한다면 `Any?`로 표현해야 한다. (하지만 제네릭의 타입 인자는 nullable하다. 그건 당연하지 타입 인자를 nullable로 받을 수 있는 거니까)
- `Unit`은 값 없음(`void`)에 대한 타입.
- `Nothing`은 더 이상 해당 코드가 진행되지 않음을 알리는 컴파일러와의 약속 타입.
- Collection<T>에 대해서 <TYPE?>?와 같은 케이스를 혼동하지 말고 잘 이해해라.
- List<T>와 같은 경우는 `read-only interface`이지 immutable일 수도 있고 mutable일 수도 있다.
    
    ```kotlin
    
    fun main() {
        val someSet: Set<Int> = setOf(1, 2, 3)
        (someSet as MutableSet).add(5)
        println(someSet) // 1, 2, 3, 5 <- ?? 읽기 전용이라며
        // 내부 구현으로 계속 들어가다보면 setOf는 LinkedHashSet을 반환하고
        // LinkedHashSet은 Set을 구현, HashSet을 상속하는데 HashSet에 add가 있음
        
        val someArrayList: List<Int> = arrayListOf(1, 2, 3)
        (someArrayList as MutableList).add(5)
        println(someArrayList) // 1, 2, 3, 5
    
        val someList: List<Int> = listOf(1, 2, 3)
        (someList as MutableList).add(5) // UnsupportedOperationException
    }
    ```
    
    <aside>
    ❓
    
    Kotlin에서 ArrayList를 선언해서 사용하는 경우에는 .add를 사용할 수 있는데, 단순히 listOf()를 사용하는 경우에는 List<> 인터페이스로 타이핑되어서 add를 사용할 수 없는 것은 알겠어. 하지만, listOf의 내부 구현을 보면 JVMUtil등을 이용해서 내부에서 ArrayList<>로 반환하게 되는데, 이건 코틀린에서 선언하는 그것과 다른거야?
    
    </aside>
    
    <aside>
    💡
    
    좋은 질문이야. Kotlin의 `listOf()`는 표면적으로는 **불변 리스트**를 반환하고, 내부적으로는 `java.util.Arrays$ArrayList` 또는 `java.util.Collections$UnmodifiableList` 같은 **Java 플랫폼의 구현체**를 사용하는 경우가 많지만, 여기서 말하는 ArrayList와 Kotlin에서 `mutableListOf()`나 `ArrayList()`로 생성한 **가변 리스트**는 명확히 다르다고 봐야 해.
    
    ---
    
    ### **✅ 핵심 차이 정리**
    
    | **구분** | listOf() | mutableListOf() **/** ArrayList() |
    | --- | --- | --- |
    | 반환 타입 | List<T> (읽기 전용 인터페이스) | MutableList<T> |
    | 내부 구현 | Java의 Arrays$ArrayList 또는 UnmodifiableList | ArrayList<T> (java.util.ArrayList 기반이거나 Kotlin 구현체) |
    | add(), remove() 가능 여부 | ❌ 불가능 (컴파일 에러) | ✅ 가능 |
    | 메모리/성능 | 작은 요소 수에 최적화 (불변성 유지) | 일반적인 동적 배열 (가변성 중심) |
    
    ---
    
    ### **📌 내부 구현은 왜 ArrayList일까?**
    
    Kotlin의 `listOf("a", "b", "c")`를 보면 내부적으로:
    
    ```kotlin
    public fun <T> listOf(vararg elements: T): List<T> =
        if (elements.isNotEmpty()) ArraysUtilJVM.listOf(elements) else emptyList()
    ```
    
    `ArraysUtilJVM.listOf(elements)`는 다음과 같이 Java의 `Arrays.asList()`를 사용해서:
    
    ```kotlin
    return Arrays.asList(elements);
    ```
    
    → 결국 **Java의 `java.util.Arrays$ArrayList`** 객체가 생성돼.
    
    하지만 이 객체는 java.util.ArrayList와 다르게 add() 등의 메서드가 **`UnsupportedOperationException`**을 던지는 읽기 전용 리스트이기 때문에 Kotlin에서는 `List<T>`로 노출되고, 가변성이 제거돼.
    
    ---
    
    ### **💡 결론**
    
    - listOf()는 내부적으로 ArrayList 유사 객체를 반환하더라도 **불변의 계약**을 유지하기 때문에 add()를 쓸 수 없어.
    - mutableListOf()나 ArrayList()는 처음부터 **가변 리스트**를 만들기 위한 것이므로 add() 가능.
    - 같은 이름이라도 **동작 컨트랙트와 반환 타입**이 다르므로, 내부 구현체가 비슷하더라도 실질적인 사용법이나 안전성은 다르게 취급해야 해.
    </aside>
    
    따라서 listOf를 통한다면 불변 리스트, 아니라면 ArrayList등 mutable한 리스트이다. 단, List<T>를 통해 interface로 닫는 경우가 생길 수 있다.
    
    → 이로 인한 Jetpack Compose의 List<T>에 대한 Unstability(불변 객체에 대해 Stable하다고 판단)로 인해 불변일 것 같은 해당 인자의 변화에 대해서도 항상 리컴포지션(리렌더링)이 일어난다.
    
    → 이를 해결하기 위한 [`kotlinx.collections.immutable`](https://github.com/Kotlin/kotlinx.collections.immutable) 라이브러리가 있다. 해당 라이브러리에서 사용하는 `PersistentList`의 경우 `add()`를 제공하지만, 이는 방어적 복사로, 새로운 리스트를 반환한다.
    
- `ConcurrentModificationException`과 인사이트
    - 이전에 알림(핸드폰 위에 나타나는 그것) 리스트 관련하여 비슷한 문제가 있었음.
    - 앱의 여러 스레드에서 동시다발적으로 해당 API를 호출하거나, 하나의 스레드에서 여러번 호출하는 경우가 생기는데 안드로이드의 경우 Binder라는 것을 이용해 IPC를 사용함(상당히 한정적인 Buffer). 이 경우 Binder의 Buffer가 터져서 뻗는 경우도 발생함.
    - 동시 수정 문제를 해결하기 위해서 Kotlin의 `Channel`(자체 MQ 같은 인터페이스)을 사용했음. → 근데 생각해보니 그냥 `persistentList`썼으면 됐잖아..?
    - 이외에 별도의 버퍼 오버플로우 문제는 디바운서로 해결(그냥 호출이 너무 잦아서 터지던 것)
- `Array`는 크기 고정 컨테이너 배열 클래스, `Iterable`은 가변 배열에 대한 인터페이스
    
    ```kotlin
    arrayOf(1,2,3).asIterable() // 가능
    arrayOf(1,2,3) as Iterable<Int> // class [Ljava.lang.Integer; cannot be cast to class java.lang.Iterable ([Ljava.lang.Integer; and java.lang.Iterable are in module java.base of loader 'bootstrap')
    ```
    

### 문제

- 다음 Kotlin 코드를 실행하면 어떤 일이 발생할까? 실행 결과와 그 이유를 설명해보자.
    
    ```kotlin
    fun main() {
        val list1: List<Int> = listOf(1, 2, 3)
        val list2: List<Int> = mutableListOf(1, 2, 3)
    
        (list1 as MutableList).add(4)
        (list2 as MutableList).add(4)
    
        println("list1 = $list1")
        println("list2 = $list2")
    }
    ```
    
- 다음 중 런타임 에러 없이 정상적으로 작동하는 코드는 무엇일까?
그 이유를 코드별로 설명해보자.
    
    ```kotlin
    val a = arrayOf(1, 2, 3)
    val b = a as Iterable<Int>  // 코드 A
    
    val c = arrayOf(1, 2, 3)
    val d = c.asIterable()      // 코드 B
    
    val e = listOf(1, 2, 3)
    val f = (e as MutableList).add(4) // 코드 C
    
    val g = arrayListOf(1, 2, 3)
    val h = (g as MutableList).add(4) // 코드 D
    ```
