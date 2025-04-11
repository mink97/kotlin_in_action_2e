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
    <details>
    - 해설
        1. 해당 배열에는 `“Height everyone”`만 포함된다. when 절은 위에서부터 아래로, 분기 문에 해당하는 경우에 한해서 단 한 번 평가한다.(switch처럼 단순 fallThrough 하지 않는다) 만약 해당하는 케이스에 대해서 모두 반영하고 싶었다면 if 문을 두번 이용해서 사용해야 한다. 생각보다 when 절을 아무 생각없이 쓰다보면 이 사실을 망각할 수 있다. ~~(어떻게 알았냐고..?)~~
        2. 타입 추론은 results에 대해 mutableList를 대입하려할 때에 발생한다. 리스트는 `MutableList<T>`, 즉 제네릭을 갖는 값이므로 컴파일타임에 정보를 파악해야한다. `mutableListOf<String>()`을 대입하는 것 말고 results 자체에 타입을 명시하고 `mutableListOf()`만 대입하는 것도 가능하다.
            
            ```kotlin
            val results = mutableListOf<String>()
            val resultsWithType: MutableList<String> = mutableListOf()
            ```  
    </details>
    
            
    
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
    <details>
    - 해설
        
        `whyPrintingMe`는 `println()`이라는 함수의 결과 값을 가지게 되는데, 이는 java의 `void`와 동일하지만서도, `Unit`이라는 별도의 코틀린 타입으로 반환된다. 즉, `println()`이 실행되어 평가된 이후의 값은 `Unit`이므로, `whyPrintingMe`는 `Unit` 값을 가지게 된다.
        
        `ifYouLoveMe`는 컴파일 에러가 발생한다. if 문을 제어문이 아닌 평가식으로 사용하는 경우에는 else 가 반드시 필요하기 때문이다. (안 그러면 else일 때는 뭔 값인데? null도 아니야 뭣도 아니야 너가 안 써놨잖아.)
        
        `dontPlayMe`는 놀랍게도 할당 가능하다. 엥? 문자열 반환하는 `try-catch` 평가식 아님? 이라고 생각할 수 있으나, catch에서 반환 값이 적혀있지 않으므로 컴파일러는 해당 try-catch를 Unit으로 간주하게 된다(반환 값이 없는 단순 함수 호출식) + 만약 catch에 string을 적어준다면, `dontPlayMe`는 `String`으로 간주된다.
        
        ```kotlin
        val dontPlayMe: Unit = try {
            "I won't"
            throw IllegalArgumentException("Sorry i will")
        } catch (e: Exception) {
            // do nothing
        }
          
         val dontPlayMe: String = try {
            "I won't"
            throw IllegalArgumentException("Sorry i will")
        } catch (e: Exception) {
            "hehehe"
        }
        ```
        
        `amIValue`는 놀랍게도 컴파일 가능하고, 값이지만 값이 아니다. 이게 무슨 말일까?, 책에서 throw는 표현식(평가식)이라고 했다. 즉, 이후에 오는 exception의 경우 java의 Throwable을 조상으로 갖는 건 동일하지만, 코틀린 내부에서 평가할 때에는 `Nothing`이라는 타입으로 간주한다. 이는 typescript 진영의 `never`와 같은 녀석으로서, 컴파일 타임에 표현식으로 사용할 수 있음과 동시에 컴파일러에게 이 이후의 코드는 절대 실행되지 않는다는 보장과, 런타임에서는 반드시 값을 가질 수 없게 되는 것을 약속한다. 따라서, 다음과 같은 것을 통해 nullable한 값을 null-safe한 값으로 간주하게 할 수 있다.
        
        ```kotlin
        val iAmNullString: String? = null
        val mustBeStringOrThrow: String = iAmNullString ?: throw IllegalArgumentException("I am null")
        println(mustBeStringOrThrow)
        
        // Exception in thread "main" java.lang.IllegalArgumentException: I am null
        ```  
    </details>
    
