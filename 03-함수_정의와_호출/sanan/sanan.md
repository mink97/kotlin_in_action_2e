### 가장 중요하다고 생각한 것

확장함수는 코틀린의 꽃이다.

- named parameter - IDE 없이도 이해할 수 있게 코드를 작성하라
    - default parameter를 지정한 경우 인자값이 named 되지 않은 호출은 전부 바뀐 값을 반영한다 - 당연한 얘기
- 함수 선언을 위해 클래스를 선언할 필요가 없다
    - 최상위 함수, 최상위 프로퍼티
        
        → java로 변환 -  알아서 클래스 선언(파일이름) 이후 내부에 static function, static value로 선언해준다.
        
- 메서드를 다른 클래스에 추가(⭐⭐⭐⭐⭐)
    
    [(함수 리터럴이라는 표현도 중요하다)](https://medium.com/depayse/kotlin-%ED%95%A8%EC%88%98-%EB%A6%AC%ED%84%B0%EB%9F%B4-function-literal-1-%EC%9D%BC%EA%B8%89-%EA%B0%9D%EC%B2%B4-first-class-citizen-44cdd64b1abe)
    
    - 확장 함수
        - 클래스의 public한 부분만 사용할 수 있다 (외부의 해당 클래스 adapter 느낌)
        - 확장 함수가 해당 수신 객체 타입의 다른 확장 함수를 호출하는 것도 가능하다. (뭔 함수든 알바가 아니다)
        - 함수는 확장 함수를 호출할 때 수신 객쳬로 지정한
        변수의 컴파일 시점의 타입에 의해 결정되지 , 실행 시간에 그 변수에 저장된 객체의
        타입에 의해 결정되지 않는다．
        - 같은 시그니처 → 멤버 > 확장
        - Join 실제 예시
          <img width="708" alt="스크린샷 2025-04-15 오전 9 12 50" src="https://github.com/user-attachments/assets/46f1a28a-3ff5-44d7-a2d4-83665690b48f" />
    - 확장 프로퍼티
        - 사실상 확장함수에 prefix로 get/set 지정해서 코틀린이 프로퍼티로 사용할 수 있는 방식인데, 과연 이게 좋은 방식일까?
- 가변 인자
    - `vararg` 키워드와 스프레드 연산자(`*` → Array  to vararg, Iterable이나 Collection 아님)만 잘 알면 된다
    - 디스트럭처링
        - 현재는 디스트럭처링되는 객체의 멤버 순서대로 (…, …)를 받지만, 이후에 K2 Compiler에서 named로 명시해야 디스트럭처링 가능하게 바꾼다고 함(아마 data class에 대한 얘기였으므로 pair는 그대로 이름 가변적으로 쓸 수 있을 듯)
- 3중 따옴표
    - @Language(”JSON”)은 히트인듯
        
        ```kotlin
        @Language("JSON")
            val json =  """
                {
                  "monday" : "${R.string.common_keywords_date_monday_short.rememberLocalize()}",
                }
            """.trimIndent()
        ```
        
- 로컬 함수 선언
    - 호출마다 재생성되는가?
        - https://stackoverflow.com/questions/78402030/are-local-variables-and-functions-in-kotlin-re-created-every-time-a-function-is
        - Kotlin의 local 함수 선언은 컴파일 타임에 정의되며, 코드에 포함된 함수 정의는 한 번만 생성됩니다. 즉, 함수 호출 시마다 새로운 함수 객체가 생성되는 것이 아니라, 이미 컴파일된 함수 정의를 호출하는 방식입니다. 다만, local 함수가 외부 변수(캡처된 변수)를 참조하면, 클로저(closure) 형태로 컴파일되며 이때 일부 오버헤드가 발생할 수 있습니다. 그러나 이 경우에도 함수 선언 자체가 호출할 때마다 재생성되지는 않고, 필요한 컨텍스트를 캡처한 래퍼 객체로 존재하게 됩니다.

---

### 문제

- UI(Presentation) 레이어에 해당하는 패키지에서 `Domain` 객체의 국제화된 이름을 연산해서 뷰에서 그려야하는 경우 어떠한 방식으로 해당 함수를 구현하고 배치하면 깔끔할 수 있을까?
- 다음 실행의 출력을 예상해보고 그 이유를 설명해보자.
    
    ```kotlin
    fun String.toString() = "hi"
    
    data class Exam(
        val name: String = UUID.randomUUID().toString().substring(0, 3)
    )
    
    fun main() {
        println(Exam().name.toString())
    }
    ```
