# 11. 제네릭스

### 11.2.3 reified 타입 파라미터를 통해 java.lang.Class 파라미터 피하기
> 안드로이드의 startActivity 함수 간단하게 만들기
> ```kotlin
> inline fun <reified T : Activity>
>     Context.startActivity() {
>   val intent = Intent(this, T::class.java)
>   startActivity(intent)
> }
>
> startActivity<DetailActivity>()
> ```

### 11.2.5 실체화된 타입 파라미터의 제약
- 사용가능
  - 타입 검사와 캐스팅(is, !is, as, as?)
  - kotlin 리플렉션 API(::class)
  - java.lang.Class 얻기(::class.java)
  - 다른 함수를 호출할 때 타입 인자로 사용
- 사용 불가
  - 타입 파라미터 클래스의 인스턴스 생성
  - 타입 파라미터 클래스의 동반 객체 메서드 호출
  - 실체화된 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
  - **실체화된 타입 파라미터는 인라인 함수에만 사용 가능**
- 























