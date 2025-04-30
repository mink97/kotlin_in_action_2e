# **컬렉션에 대한 함수형 API**

## **filter & map**

- `filter { 조건 }`: 조건을 만족하는 **원소만 추출**
- `map { 변환 }`: 각 원소를 **다른 형태로 변환**

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))

val over30 = people.filter { it.age >= 30 }
// → [Person(name=Bob, age=31)]

val names = people.map { it.name }
// → [Alice, Bob]
```

- **함수 연결 가능**

```kotlin
people.filter { it.age > 30 }.map(Person::name)
// → [Bob]
```

- `filterIndexed`, `mapIndexed`: 인덱스를 활용하는 버전

## **reduce & fold**

- `reduce { acc, element -> ... }`: 첫 원소부터 누적 시작 (빈 리스트 불가)
- `fold(initial) { acc, element -> ... }`: **초깃값 지정 가능**

```kotlin
val list = listOf(1, 2, 3, 4)

list.reduce { acc, e -> acc + e }     // → 10
list.fold("") { acc, p -> acc + p }   // 문자열 누적
```

- `runningReduce`, `runningFold`: **중간값 추적 가능**

```kotlin
list.runningReduce { acc, e -> acc + e }
// → [1, 3, 6, 10]
```

## **all, any, none, count, find**

- `all { 조건 }`: 모두 만족하는가
- `any { 조건 }`: 하나라도 만족하는가
- `none { 조건 }`: 전부 만족하지 않는가
- `count { 조건 }`: 만족하는 원소 수
- `find { 조건 }`: 조건 만족 **첫 원소 반환 (없으면 null)**

```kotlin
val canBeInClub27 = { p: Person -> p.age <= 27 }

people.all(canBeInClub27)   // → false
people.any(canBeInClub27)   // → true
people.none { it.age > 100 } // → true
people.count(canBeInClub27) // → 1
people.find(canBeInClub27)  // → Person("Alice", 27)
```

## **partition**

- 조건에 따라 리스트를 **2개로 나눔** (참 / 거짓)

```kotlin
val (under27, over27) = people.partition(canBeInClub27)
```

## **groupBy**

- 조건에 따라 **여러 그룹의 맵으로 변환**

```kotlin
people.groupBy { it.age }
// → Map<Int, List<Person>>
```

```kotlin
listOf("apple", "banana", "apricot").groupBy(String::first)
// → {a=[apple, apricot], b=[banana]}
```

## **associate / associateBy / associateWith**

- 리스트를 **Map으로 변환**

```kotlin
people.associate { it.name to it.age }
// → {Alice=29, Bob=31}

people.associateWith { it.age }
// → {Person(...)=29, ...}

people.associateBy { it.age }
// → 마지막 키만 남음 (같은 키 중복 제거됨)
```

## **replaceAll & fill**

- `replaceAll { 람다 }`: 리스트 내 값 전체 교체
- `fill("value")`: 동일한 값으로 모두 채움

## **ifEmpty / ifBlank**

- 비어 있는 컬렉션이나 공백 문자열을 **기본값으로 대체**

```kotlin
emptyList<String>().ifEmpty { listOf("default") }

"  ".ifBlank { "unnamed" }
```

## **chunked & windowed**

- `chunked(n)`: 고정 크기만큼 나눔 (겹치지 않음)
- `windowed(n)`: 슬라이딩 윈도우 (겹침 가능)

```kotlin
listOf(1, 2, 3, 4, 5).chunked(2)
// → [[1, 2], [3, 4], [5]]

list.windowed(3)
// → [[1,2,3], [2,3,4], [3,4,5]]
```

## **zip**

- 두 컬렉션을 **인덱스 기준으로 Pair로 합침**

```kotlin
val names = listOf("Joe", "Mary")
val ages = listOf(22, 31)

names.zip(ages) // → [(Joe,22), (Mary,31)]
names zip ages // → [(Joe,22), (Mary,31)] (중위 표현식 가능)
```

## **flatMap / flatten**

- **중첩 리스트를 펼치거나 변환 후 펼침**

```kotlin
val library =listOf(
        Book("Kotlinin Action", listOf("Isakova","Elizarov","Aigner","Jemerov")),
        Book("Atomic Kotlin",listOf("Eckel","Isakova")),
        Book("The Three-BodyPrblem", listOf("Liu"))
    )

val authors = library.map { it.authors } // [[Isakova, Elizarov, Aigner, Jemerov], [Eckel, Isakova], [Liu]]
// → List<List<String>>

library.flatMap { it.authors } // [Isakova, Elizarov, Aigner, Jemerov, Eckel, Isakova, Liu]
// → List<String>

authors.flatten() // [Isakova, Elizarov, Aigner, Jemerov, Eckel, Isakova, Liu]
// → List<String>
```

# **시퀀스 (Sequences)**

## **즉시 계산 vs 지연 계산**

- 일반 컬렉션 연산 → **각 단계마다 리스트 생성**
- 시퀀스 연산 → **원소 단위로 연산**, 중간 컬렉션 없음

```kotlin
people.asSequence() // 원본 컬렉션을 시퀀스로 변환
      .map { it.name }
      .filter { it.startsWith("A") }
      .toList() // 메소드 체이닝을 마치고, 최종 시퀀스를 다시 컬렉션으로 변환
```

## **중간 연산 vs 최종 연산**

![image](https://github.com/user-attachments/assets/9ba877a3-6c06-4e03-bd3d-b5a658899bca)


- **중간 연산**: `map`, `filter` → 시퀀스 반환 (지연됨)
- **최종 연산**: `toList`, `sum`, `find`, `forEach` → **계산 발생**
    - 하나씩 처리되기에, 이미 답을 찾은 경우 불필요한 추가 연산이 발생하지 않을 수 있다.

> map → filter → toList()가 없으면 아무 계산도 일어나지 않음
> 

## **시퀀스 최적화 효과**

![image](https://github.com/user-attachments/assets/e3a60f17-be42-4303-974f-f5f6acc4adf8)


```kotlin
// 순서 바꾸기만 해도 성능 차이 남
people.asSequence()
  .filter { it.name.length < 4 }
  .map { it.name }
  .toList()
```

> filter 먼저 하면 불필요한 map 줄임 → 효율 ↑
> 

## **generateSequence**

- 무한 시퀀스 생성

```kotlin
val naturals = generateSequence(0) { it + 1 } // 여기서는 계산이 이루어지지 않음.
val under100 = naturals.takeWhile { it <= 100 } // 여기서는 계산이 이루어지지 않음.
under100.sum()  // → 5050 // sum의 결과를 계산할 때, 지연 계산이 수행.
```

## **계층 순회 예시**

```kotlin
generateSequence(file) { it.parentFile }
  .any { it.isHidden }
```

> 파일의 상위 디렉토리 중 숨김 디렉토리 있는지 검사
> 

# 요약 핵심 포인트

| 분류 | 기능 | 대표 함수 | 설명 |
| --- | --- | --- | --- |
| 🔍 필터링 | 조건에 따라 원소 선택 | `filter`, `filterNot`, `filterIndexed` | 술어(`predicate`)에 따라 원소를 선택/제외 |
| 🔁 변환 | 원소를 다른 형태로 변환 | `map`, `mapIndexed` | 각 원소에 변환 함수 적용 |
| 📊 종합 | 누적값 계산 | `reduce`, `fold`, `runningReduce`, `runningFold` | 누산기(`accumulator`)를 이용해 리스트를 하나의 값으로 통합 |
| 🔎 조건 검사 | 불변 조건 확인 | `all`, `any`, `none`, `count`, `find` | 술어에 따라 모든/일부 원소 조건 검사 또는 개수/첫 원소 반환 |
| 👥 그룹화 | 조건별로 묶기 | `groupBy`, `partition` | 조건을 기준으로 2개 또는 여러 그룹으로 나눔 |
| 🗺 맵 변환 | 컬렉션 → Map | `associate`, `associateBy`, `associateWith` | key/value 쌍을 정의해 맵 생성 |
| 🧱 하위 분할 | 연속 혹은 분할 리스트 생성 | `chunked`, `windowed` | 고정 크기 청크 또는 슬라이딩 윈도우 생성 |
| 🔗 결합 | 리스트 병합 | `zip` | 두 리스트를 인덱스로 짝지어 Pair 생성 |
| 📦 내포 해제 | 중첩 컬렉션 해제 | `flatten`, `flatMap` | 리스트<리스트>를 평탄화하거나 변환 후 평탄화 |
| ⚙️ 가변 처리 | 원소 일괄 수정 | `replaceAll`, `fill` | 리스트 전체를 람다 결과로 치환하거나 동일 값으로 채움 |
| 🕳 빈 처리 | 빈 리스트/문자열 대체 | `ifEmpty`, `ifBlank` | 비었을 때 기본값으로 대체 |
| ⏱ 지연 연산 | 중간 결과 없이 효율적 연산 | `asSequence`, `generateSequence` | 중간 리스트 생성 없이 연산을 지연 처리 |
| 🔚 시퀀스 실행 | 최종 결과 반환 | `toList`, `sum`, `find`, `forEach` | 시퀀스에서 계산을 실제로 수행하는 종료 연산 |

# 오늘의 퀴즈

## 문제 1번

```kotlin
val numbers = emptyList<Int>()

val result1 = numbers.all { it > 100 }
val result2 = numbers.any { it > 100 }
val result3 = numbers.none { it > 100 }

println("$result1, $result2, $result3")
```

**Q. 출력되는 결과로 올바른 것은 무엇인가?**

A. `false, false, true`

B. `true, false, true`

C. `false, false, false`

D. `true, true, false`



## 문제 2번

```kotlin
val people = listOf(
    Person("Ann", 25),
    Person("Bob", 30),
    Person("Charlie", 35)
)

people
    .asSequence()
    .map { println("map: ${it.name}"); it.name }
    .filter { println("filter: $it"); it.length < 4 }
    .toList()
```

### Q. 위 코드에서 출력 순서를 고르시오 (올바른 순서로 **출력되는 println 순서** 기준)

A. 

```kotlin
map: Ann
map: Bob
map: Charlie
filter: Ann
filter: Bob
filter: Charlie
```

B. 

```kotlin
filter: Ann
filter: Bob
filter: Charlie
map: Ann
map: Bob
map: Charlie
```

C. 

```kotlin
map: Ann
filter: Ann
map: Bob
filter: Bob
map: Charlie
filter: Charlie
```

D. 

```kotlin
filter: Ann
map: Ann
filter: Bob
map: Bob
filter: Charlie
map: Charlie
```


## 문제 3번

```kotlin
val hugeList = List(300_000_000) { it }

fun main() {

    val evens1 = hugeList.filter { it % 2 == 0 }
    println(evens1.size) // ?
}
```

- 출력 결과는 무엇일까?
