- 요약
    - 컬렉션 원소를 직접 순회하는 대신, 표준 라이브러리 함수와 람다를 조합해 대부분의 일반적인 연산을 처리할 수 있다.
    - filter, map 함수는 컬렉션을 다루는 기본으로,
        - filter: 주어진 술어에 따라 컬렉션 원소를 걸러낸다.
        - map: 컬렉션 원소를 새로운 형태로 변환한다.
    - reduce와 fold 연산을 사용하면 컬렉션으로부터 정보를 종합할 수 있으며,
        - 컬렉션 원소들로부터 하나의 값을 계산해낼 수 있다.
    - associate, groupBy 함수를 사용하면 평평한 리스트를 맵으로 변환하여,
        - 자신의 기준에 따라 데이터를 구조화할 수 있다.
    - 데이터 컬렉션이 인덱스와 연관된 경우에는
        - chunked, windowed, zip 함수를 사용해
        - 하위 그룹을 만들거나 여러 컬렉션을 하나로 합칠 수 있다.
    - 불린(Boolean)을 반환하는 람다 함수(술어)를 사용해
        - all, any, none과 같은 함수로
        - 컬렉션에 대해 어떤 불변 조건이 성립하는지 검사할 수 있다.
    - 내포된 컬렉션을 처리할 경우
        - flatten 함수를 통해 내포된 원소를 꺼낼 수 있다.
        - flatMap을 사용하면 변환(map)과 펼쳐내기(flatten)를 한꺼번에 수행할 수 있다.
    - 시퀀스(Sequence)를 사용하면
        - 중간 결과를 담는 컬렉션을 생성하지 않고도
        - 컬렉션에 대한 여러 연산을 지연 계산하여 조합할 수 있다.
    - 컬렉션을 조작하기 위해 사용한 함수들은 시퀀스에도 그대로 적용할 수 있다.
- 계산을 반복하지 마라. 불합리한 계산식이 될 수 있는지 명심할 것 (특히 아이템이 많은 경우)
- runningReduce / fold
- 술어를 포함한 count를 명심
- windowed → 슬라이딩 윈도우 / chunked → 서로소 집합
- windowed 사용시, window size > size인 경우 빈 배열 반환
- chunked  용례
    
    대량 연산 비동기처리 (마치 Promise.all)
    
    ```kotlin
    // response는 List<DTO>, processProducts는 List<DTO>에 대한 연산
    coroutineScope {
    	response.chunked(10).map { chunk ->
    	    async { processProducts(chunk) }
    	}.awaitAll()
    }
    ```
    
- sequence와 람다 연산
    
    ```kotlin
    data class Person(val age: Int)
    
    fun logTimeLapsed(label: String = "", block: () -> Unit) {
        val start = System.currentTimeMillis()
        block()
        val end = System.currentTimeMillis()
        println("$label Time elapsed: ${end - start} ms")
    }
    
    // 나이가 홀수이자 세자릿수인 사람을 찾는 케이스
    // 그냥 filter만 / take를 이용한 케이스에 대한 소요시간
    fun main() {
        val people = (1..100000000).map { Person(it) }
        logTimeLapsed("Eager with multiple filters") {
            val result = people
                .filter { it.age % 2 != 0  }
                .filter { it.age in 100..999 }
              //.take(10)
        } // 757ms, 767ms
    
        logTimeLapsed("Eager with one filter") {
            val result = people
    							        .filter { it.age % 2 != 0  && it.age in 100..999 }
                        //.take(10)
        } // 145ms, 126ms
    
        logTimeLapsed("Lazy with multiple filters") {
            val result = people.asSequence()
                .filter { it.age % 2 != 0  }
                .filter { it.age in 100..999 }
              //.take(10)
                .toList()
        } // 545ms, 5ms
        
        logTimeLapsed("Lazy with one filter") {
            val result = people.asSequence()
                .filter { it.age % 2 != 0  && it.age in 100..999 }
              //.take(10)
                .toList()
        } // 428ms, 0ms
    
    }
    ```
    
- 문제 스킵
