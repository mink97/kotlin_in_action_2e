## 가장 중요하다고 생각한 부분

**코틀린에서는 (자바는 함수형 인터페이스로 피똥쌌지만) 함수가 일급 객체다**

- if () a else b 방식의 평가식- 값 반환
- 타입 추론
    
    block function말고, expression function의 경우에 해당한다
    
    → 함수 블럭자체가 아닌, 람다로서의 함수를 대입하는 식의 경우. 이는 값을 대입하는 부분과 같으므로 타입 추론이 자연스럽게 이뤄진다고 이해하면 편할 것 같다.
    
    개인적으로 expression function이 코틀린스럽다고 생각하지만 명료한 프로그래밍을 위해서(+ 함수가 더 복잡한 경우) block function이 더 괜찮다고 생각하는 편이다.
    
- 문자열 템플릿은 내부적으로 toString()을 사용한다.
- 코틀린에서 property의 이름을 갖는 get, set이 prefix인 함수는 코틀린 자체에서 생성하는 게터, 세터를 섀도잉한다. (컴파일 오류 뜰 때도 있음)
- 커스텀 getter, setter 혹은 private set등을 편하게 사용할 수 있음.
    
    커스텀 getter, setter는 주의해서 사용하는 것이 좋다. 개인적으로 계산식이 복잡해지면 커스텀 게터보다는 그냥 get??()로 함수화로 나타내는 편 - 인지적으로 해당 게터가 복잡한 연산을 할 거라고 생각하지 않을 가능성이 높으니.
    

### 문제

- 다음 상황의 출력문을 유추해보고, 타입 추론 지점을 찾아보자.
    
    ```kotlin
    data class Canvas(
        val width: Int = 0,
        val height: Int = 0,
    ) {
        fun sayHeight(): String {
            return "Height everyone"
        }
    
        fun sayWidth(): String {
            return "Dance width me"
        }
    }
    
    fun main() {
        val canvas = Canvas(100, 200)
    
        val results = mutableListOf<String>()
    
        when {
            canvas.height >= 100 -> results.add(canvas.sayHeight())
            canvas.width >= 100 -> results.add(canvas.sayWidth())
            else -> results.add("WTF")
        }
    
        println(results)
    }
    
    ```
    
            
    
- 다음 중 컴파일 에러가 발생하지 않는 것은 무엇일까?
    
    ```kotlin
    val whyPrintingMe = println("Be quiet")
    
    val ifYouLoveMe = if (true) "Love you"
    
    val dontPlayMe = try {
        "I won't"
        throw IllegalArgumentException("Sorry i will")
    } catch (e: Exception) {
        // do nothing
    }
    
    val amIValue = throw IllegalArgumentException("No you're not")
    ```
    
