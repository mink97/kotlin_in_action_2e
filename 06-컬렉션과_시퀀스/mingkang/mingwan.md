# 6. 컬렉션과 시퀀스

> 함수형 스타일로 컬렉션 다루기<br/>
> 시퀀스: 컬렉션 연산을 지연시켜 수행하기


- 일반적인 컬렉션 접근 패턴 학습
- 시퀀스에 대해 학습
- 컬렉션 연산을 즉시 실행하는 방법과 지연 실행하는 방법 비교

## 6.1 컬렉션에 대한 함수형 API
- 함수형 프로그래밍 스타일은 컬렉션을 다룰 때 여러가지 장점 제공
- 표준 라이브러리가 제공하는 함수 활용, 람다를 인자로 전달해 그 함수들의 동작 커스텀화

### 6.1.1 원소 제거와 변환: filter, map
- filter: 컬렉션을 순회하면서 주어진 람다가 true를 반환하는 원소들만 모음.
  - `list.filter {it % 2 == 0}` -> 짝수만 남김
  - `people.filter { it.age >= 30 }` -> 30살 이상인 사람들
  - 주어진 술어와 일치하는 원소들로 이뤄진 새 컬렉션을 만들 수 있지만 그 과정에서 원소를 변환하지는 않음.
- map: 입력 컬렉션의 원소를 변환. 주어진 함수를 컬렉션의 각 원소에 적용하고 그 결괏값들을 새 컬렉션에 모아줌.
  - `list.map { it * it }` -> 숫자 리스트를 제곱 리스트로 변환
  - 결과는 같은 개수의 원소가 들어있는 새 컬렉션
- 호출을 쉽게 연결 가능(filter, map...)
  - `people.filter { it.age > 30 }.map(Person::name)`
> [!NOTE]
> 리스트에서 가장 나이 많은 사람의 이름 찾기
> ```kotlin
> people.filter {
>   val oldestPerson = people.maxByOrNull(Person::age)
>   it.age == oldestPerson?.age
> }
> ```
> - 이 코드는 oldestPerson 찾는 작업을 계속 반복(불합리)
> ```kotlin
> val maxAge = people.maxByOrNull(Person::age)?.age
> people.filter { it.age == maxAge }
> ```
> - 꼭 필요하지 않은 경우 굳이 계산을 반복하지 말라.
> - 겉으로 볼 때는 단순해 보이는 식이 내부 로직의 복잡도로 인해 실제로는 엄청나게 불합리한 계산식이 될 때가 있음.
> - 항상 작성하는 코드로 인해 어떤 일이 벌어질지 명확히 이해하자.

- filterIndexed, mapIndexed: 원소의 인덱스와 원소 자체를 함께 제공
  ```kotlin
  fun main() {
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7)
    val filtered = numbers.filterIndexed { index, element ->
      index % 2 == 0 && element > 3
    }
    println(filtered)
    // [5, 7]
    val mapped = numbers.mapIndexed { index, element ->
      index + element
    }
    println(mapped)
    // [1, 3, 5, 7, 9, 11, 13]
  }
  ```
- filter 함수와 map 함수를 맵에 적용도 가능
  - filterKeys와 mapKeys는 키를 걸러내거나 변환,
  - filterValues와 mapValues는 값을 걸러내거나 변환

### 6.1.2 컬렉션 값 누적: reduce, fold
- 컬렉션의 정보를 종합하는데 사용
- 작성한 람다는 각 원소에 대해 호출되면 새로운 누적값을 반환
- reduce는 컬렉션의 첫 번째 값을 누적기에 넣음(빈 컬렉션x)
  ```kotlin
  fun main() {
    val list = listOf(1, 2, 3, 4)
    println(list.reduce { acc, element ->
      acc + element
    })
    // 10
  }
  ```
- fold는 임의의 시작 값 선택 가능
  ```kotlin
  fun main() {
    val people = lsitOf(
      Person("Alex", 29),
      Person("Natalia", 28)
    )
    val folded = people.fold("") { acc, person ->
      acc + person.name
    }
    println(folded)
  }
  //AlexNatalia
  ```
- runningReduce, runningFold는 중간 단계의 모든 누적 값을 뽑아냄(리스트 반환)

### 6.1.3 컬렉션에 술어 적용: all, any, none, count, find
- all : 모든 원소가 만족
- any : 하나라도 만족(!all == any)
- none : 하나도 만족 X(!any == none)
- count : 조건을 만족하는 원소의 개수 반환
  - count가 filter로 필터링하고 size가져오는 경우보다 효율적
- find : 조건을 만족하는 첫 번째 원소 반환
  - 원소가 전혀 없는 경우 null 반환

> [!NOTE]
> ### 술어와 빈 컬렉션의 관계
> any : false 반환
> none : true 반환
> all : true 반환(공허한 참)

### 6.1.4 리스트를 분할해 리스트의 쌍으로 만들기: partition
- 컬렉션을 어떤 술어를 만족하는 그룹과 그렇지 않은 그룹으로 나눌 필요가 있을 때
  - filter + filterNot
  ```kotlin
  data class Person(val name: String, val age: Int)
  
  val canBeInClub27 = { p: Person -> p.age <= 27 }

  fun main() {
    val people = listOf(
      Person("Alice", 26),
      Person("Bob", 29),
      Person("Carol", 31)
    )

    val (comeIn, stayOut) = people.partition(canBeInClub27) // 구조 분해 선언
    println(comeIn)
    // [Person(name=Alice, age=26]
    println(stayOut)
    // [Person(name=Bob, age=29), Person(name=Carol, age=31)]
  }
  ```
![image](https://github.com/user-attachments/assets/82bc9c17-99ef-402e-927c-eb113ca33a6e)

### 6.1.5 리스트를 여러 그룹으로 이뤄진 맵으로 바꾸기: groupBy
- 컬렉션의 원소를 어떤 특성에 따라 여러 그룹으로 나누고 싶을 때, 맵이 결과로 반환
- 컬렉션의 원소를 구분하는 특성이 *키*, 키 값에 따른 각 그룹이 *값*
- 각 그룹은 리스트에 저장
![image](https://github.com/user-attachments/assets/893c6f1c-b2ce-4622-9edc-73bc710ffd52)
```kotlin
fun main() {
  val list = listOf("apple", "apricot", "banana", "cantaloupe")
  println(lsit.groupBy(String::first))
  // {a=[apple, apricot], b=[banana], c=[cantaloupe]}
}
```

### 6.1.6 컬렉션을 맵으로 변환: associate, associateWith, associateBy
- 원소를 그룹화하지 않으면서 컬렉션으로부터 맵 생성
  ```kotlin
  fun main() {
    val people = listOf(Person("Joe", 22), Person("Mary", 31))
    val nameToAge = people.associate { it.name to it.age }
    println(nameToAge)
    // {Joe=22, Mary=31}
    println(nameToAge["Joe"])
    // 22
  }
  ```
- associateWith, associateBy : 키와 커스텀 값의 쌍을 만들어내는 대신, 컬렉션의 원소와 다른 어떤 값 사이의 연관 만들기
  - associateWith : 컬렉션의 원래 원소를 키, 람다가 만들어내는 결과는 값
  - associateBy : 람다가 만들어내는 결과를 키, 컬렉션의 원래 원소를 값
- 변환 함수가 키가 같은 값을 여러 번 추가하게 됨면 마지막 결과가 그 이전에 들어간 결과 덮어씀.

### 6.1.7 **가변 컬렉션**의 원소 변경: replaceAll, fill
- replaceAll: 람다를 통해 얻은 결과로 컬렉션의 모든 원소 변경
- fill: 가변 리스트의 모든 원소를 똑같은 값으로 변경

### 6.1.8 컬렉션의 특별한 경우 처리: ifEmpty
- 아무 원소도 없을 때 기본값 생성하는 람다 제공
- ifBlank: 공백만 있을 때

### 6.1.9 컬렉션 나누기: chunked, windowed
- 데이터를 연속적인 시간의 값들로 처리하고 싶은 경우
- windowed: 슬라이딩 윈도우 생성하고자 할 때
  - `temperatures.windowed(3)`
  - `temperatures.windowed(3) {it.sum() / it.size }`
  ![image](https://github.com/user-attachments/assets/26d75636-fa76-4f0a-8f6d-956d9331764a)
- chunked: 컬렉션을 어떤 주어진 크기의 서로 겹치지 않는 부분으로 나누고 싶을 때
  ![image](https://github.com/user-attachments/assets/2ba62939-73b0-4d39-92e6-f69f02213e98)
  - 마지막으로 만들어진 청크의 크기는 더 작음.

### 6.1.10 컬렉션 합치기: zip
- zip 함수를 사용해 두 컬렉션에서 같은 인덱스에 있는 원소들의 쌍으로 이뤄진 리스트 생성 가능
- 결과 컬렉션의 길이는 두 입력 컬렉션 중 더 짧은 쪽의 길이와 같음.
- zip 함수도 중위 표기법으로 호출 가능(but, 람다 전달 불가)
  `println(names zip ages)`
- zip을 여러번 호출하면 내포된 쌍의 리스트가 생김. `ex. ((A, B), C)`
![image](https://github.com/user-attachments/assets/64cd115f-4d33-4997-95ae-0587e2685c1b)

### 6.1.11 내포된 컬렉션의 원소 처리: flatMap과 flatten
- flatMap: 라이브러리의 모든 저자의 집합을 계산하되 추가적인 내포 없이 계산 가능
  - 우선 컬렉션의 각 원소를 파라미터로 주어진 함수를 사용해 변환
  - 변환한 결과를 하나의 리스트로 합침.
  - toSet() 호출은 결과 컬렉션에서 중복 제거
- flatten: 변환할 것이 없고 단지 컬렉션의 컬렉션을 평평한 컬렉션으로 만들기

## 6.2 지연 계산 컬렉션 연산: 시퀀스
- 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담음
- **시퀀스**는 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄하는 방법 제공
  ```kotlin
  people
    .asSequence()                     // 원본 컬렉션을 시퀀스로 변환
    .map(Person::Name)
    .filter { it.startsWith("A") }
    .toList()                         // 결과 시퀀스를 다시 리스트로 변환
  ```
- 중간 결과를 저장하는 컬렉션이 생기지 않기 때문에 원소가 많은 경우 성능이 눈에 띄게 좋아짐
- Sequence 인터페이스에서 시장
  - iterator라는 하나의 메서드만 들어있음.(시퀀스에서 원소 값 얻을 수 있음.)
- 시퀀스의 원소는 필요할 때 지연 계산
- 중간 처리 결과를 저장할 컬렉션을 만들지 않고도 연산을 연쇄적으로 적용, 연쇄적인 연산을 효율적으로 수행
- asSequence 확장 함수를 호출하면 어떤 컬렉션이든 시퀀스로 변경 가능
- 리스트로 바꿀 때는 toList 사용

> [!NOTE]
> 큰 컬렉션에 대해 연산을 연쇄시킬 때는 시퀀스를 사용하는 것을 규칙으로 삼아라.
> 10.2절에서는 중간 컬렉션을 생성함에도 코틀린에서 즉시 계산 컬렉션에 대한 연산이 더 효율적인 이유를 셜명한다.
> 하지만 컬렉션에 들어있는 원소가 많으면 중간 원소를 재배열하는 비용이 커지기 때문에 지연 계산이 더 낫다．

### 6.2.1 시퀀스 연산 실행: 중간 연산과 최종 연산
- **중간 연산**은 다른 시퀀스 반환.
  - 최초 시퀀스의 원소를 변환하는 방법을 앎.
  - 항상 지연 계산(최종 연산이 없으면 아무 내용도 적용 x)
- **최종 연산**은 *결과*를 반환.
  - *결과*: 최초 컬렉션에 대해 변환을 적용한 시퀀스에서 일련의 계산을 수행해 얻을 수 있는 컬렉션이나 원소, 수, 또는 다른 객체
![image](https://github.com/user-attachments/assets/da6636bd-6660-4e0c-91a7-a6e5e9397c45)
- 시퀀스의 경우 모든 연산은 각 원소에 대해 순차적으로 적용
- 원소에 연산을 차례대로 적용하다가 결과가 얻어지면 그 이후의 원소에 대해서는 변환이 이뤄지지 않을 수도 있다.
  ![image](https://github.com/user-attachments/assets/de37f6e4-03e4-4f63-bfa2-cfa17da28ba4)
- 컬렉션에 대해 수행하는 연산의 순서도 성능에 영향
  ![image](https://github.com/user-attachments/assets/2fda0f53-201c-4946-b759-c2f48c493700)

### 6.2.2 시퀀스 만들기
1. asSequence() 호출
2. generateSequence() 호출
   - 이전의 원소를 인자로 받아 다음 원소 계산
   ```kotlin
   fun main() {
     val naturalNumbers = generateSequence(0) { it + 1 }
     val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
     println(numbersTo100.sum()) // 5050
   }
   ```
- 시퀀스를 사용하는 일반적인 용례 중 하나는 객체의 조상들로 이뤄진 시퀀스를 만들어내는 것
  ```kotlin
  import java.io.File

  fun File.isInsideHiddenDirectory() =
  	generateSequence(this) { it.parentFile }.any { it.isHidden }
  fun main() {
  	val file = File("/Users/svtk/.HiddenDir/a.txt")
  	println(file.isInsideHiddenDirectory())
  	// true
  }
  ```
  - any를 find로 바꾸면 원하는 디렉터리를 찾을 수도 있음.

## 중요한 것 같은 부분
- 함수형 API, 위에서 설명한 여러 함수들 숙지
- 람다 내에서 중복 계산으로 불필요한 계산 늘릴 수 있으니 조심
- 시퀀스 지연연산

## 문제
1. 다음 코드를 최적화하시오.
```kotlin
data class Student(val name: String, val grade: Int)

fun topStudentsNames(students: List<Student>): List<String> {
    return students.filter { it.grade == students.maxByOrNull { s -> s.grade }?.grade }
                   .map { it.name }
}
```

2. 시퀀스 사용하는 코드로 변경해보기
```kotlin
data class Article(val title: String, val content: String)

fun getKeywordTitles(articles: List<Article>, keyword: String): List<String> {
    return articles.map { it.title }
                   .filter { it.contains(keyword) }
}
```

## 답
1.
```kotlin
  fun topStudentsNames(students: List<Student>): List<String> {
  val maxGrade = students.maxByOrNull { it.grade }?.grade
  return students.filter { it.grade == maxGrade }
                 .map { it.name }
  }
```
2.
```kotlin
  fun getKeywordTitles(articles: List<Article>, keyword: String): List<String> {
    return articles.asSequence()
                   .filter { it.title.contains(keyword) }
                   .map { it.title }
                   .toList()
  }
```
