# 06장 - (p.277 - p.)

## 컬렉션에 대한 함수 API

### 원소 제거와 변환: `filter` 와 `map`

- `filter`: 컬렉셕을 순회하며 주어진 람다가 true 를 반환하는 원소들만 모음

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
	val list = listOf(1, 2, 3, 4)
	println(list.filter { it % 2 == 0 })
	// [2, 4]
}
```

- 주어진 술어를 만족하는 원소들로만 이뤄진 새로운 컬렉션 반환

- `map`: 입력 컬렉션의 원소를 변환. 주어진 함수를 컬렉션의 각 원소에 적용하고 그 결괏값들을 새 컬렉션에 모음

- 걸러내거나 변환하는 연산이 원소의 값 뿐만 아니라, 인덱스에 따라서도 달라진다면 `filterIndexed` 와 `mapIndexed` 를 사용

```kotlin
fun main() {
  val numbers = listOf(1, 2, 3, 4, 5, 6, 7)
  val filtered  numbers.filterIndexed { index, element ->
	  index % 2 == 0 && element & 3
  }
  println(filtered)
  // [5, 7]
}
```

### 컬렉션 값 누적: `reduce` 와 `fold`

- `reduce`
  - 컬렉션의 첫 번째 값을 누적기에 넣음
  - 그 후 람다가 호출되면서 누적 값과 2번째 원소가 인자로 전달됨

```kotlin
fun main() {
	val list = listOf(1, 2, 3, 4)
	println(list.reduce { acc, element ->
		acc + element
	})
	// 10
}
```

- `fold`

  - `reduce` 와 비슷하지만, 컬렉션 첫 번째 원소를 누적 값으로 시작하는 대신, 임의의 시작 값을 선택 가능

- reduce 나 fold 에서 중간 단계의 모든 누적 값을 뽑아내고 싶다면 runningReduce 와 runningFold 사용
  - 유일한 차이점은 reduce 나 fold 는 결괏값을 하나만 반환하지만, runningReduce 와 runningFold 는 리스트를 반환함
  - 이 리스트에는 최종 결과 (마지막 원소) 와 함께, 모든 중간 누적 값이 들어있음

### 컬렉션에 술어 적용: all, any, none, count, find

- `count`: 조건을 만족하는 원소의 개수를 반환
- `find`: 조건을 만족하는 첫 번째 원소를 반환
- `all`: 모든 원소가 해당 술어를 만족하는지
- `any`: 술어를 만족하는 원소가 하나라도 있는지
  - 어떤 조건에 대해 `!all` 을 수행한 결과와, 그 조건의 부정에 대해 `any` 를 수행한 결과는 같다
    (당연하다)
- `none`: 모든 원소가 술어를 만족하지 않는지

### 리스트를 분할해 리스트의 쌍으로 만들기: `partition`

- 컬렉션을 어떤 술어를 만족하는 그룹과 그렇지 않은 그룹으로 나눌 필요가 있을 때, 두 그룹이 모두 필요하다면 `filter` 와 `filterNot` 을 사용할 수 있음
- `partition` 을 사용하면 이를 더 간단히 처리 가능

```kotlin
fun main() {
  val people = listOf(
	  Perosn("Alice", 26),
	  Person("Bob", 29),
	  Person("Carol", 31)
  )
  val comeIn = people.filter(canBeInClub27)
  val stayOut = people.filterNot(canBeInClub27)

  val (comeIn, stayOut) = people.partition(canBeInClub27)
}

```

### 리스트를 여러 그룹으로 이뤄진 맵으로 바꾸기: `groupBy`

```kotlin
fun main() {
  val people = listOf(
	  Perosn("Alice", 31),
	  Person("Bob", 29),
	  Person("Carol", 31)
  )
  println(people.groupBy { it.age })
  // {31=[Person(name=Alice, age=31), Perseon(name=Carol, age=31)], 29=[Person(name=Bob, age=29)]}
}
```

- 각 그룹은 리스트에 저장되므로, 따라서 결과의 타입은 `Map<Int, List<Person>>`

### 컬렉션을 맵으로 변환: `associate`, `associateWith`, `associateBy`

- 원소를 그룹화하지 않으면서 컬렉션으로부터 맵을 만들어내고 싶다면, `associate` 함수를 사용

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

- 위 예제는 `associate` 함수를 사용해 Person 의 리스트를 이름과 나이를 연관시켜주는 맵으로 변환한 다음, 다른 모든 `Map<String, Int>` 타입의 맵과 마찬가지로 예제 값을 맵에서 찾아봄

- `asscociateWith`: 컬렉션의 원래 원소를 키로 사용
- `associateBy`: 컬렉션의 원래 원소를 맵의 값으로 하고, 람다가 만들어내는 값을 맵의 키로 사용

### 가변 컬렉션의 원소 변경: `replaceAll`, `fill`

### 컬렉션의 특별한 경우 처리: `ifEmpty`

### 컬렉션 나누기: `chunked` 와 `windowed`

### 컬렉션 합치기: `zip`

### 내포된 컬렉션의 원소 처리: `flatMap` 과 `flatten`

### 지연 계산 컬렉션 연산: 시퀀스

- 컬렉션 함수를 연쇄하면 매 단계마다 계산 중관 결과를 새로운 컬렉션에 임시로 담음
- 시퀀스는 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄하는 방법을 제공

```kotlin
people
	.asSequence()
	.map(Person::name)
	.filter { it.startsWith("A") }
	.toList()
```

- 시퀀스의 원소는 필요할 때 lazy evaluation 됨
- 큰 컬렉션에 대해 연산을 연쇄할 때는 시퀀스를 사용하는 것을 추천

  - 중간 컬렉션을 생성함에도 코틀린에서 즉시 계산 컬렉션에 대한 연산이 더 효율적이지만, 원소가 많으면 중간 원소 재배열 비용이 커지기 때문에 지연 계산이 더 나음

- 시퀀스의 경우, 모든 연산은 각 원소에 대해 순차적으로 적용됨

  - 즉, 첫 번째 원소가 처리되고, 다시 두 번째 원소가 처리되며, 이런 처리가 모든 원소에 반복됨

- `asSequnce()` 를 사용해 시퀀스를 만들 수 있고, `generateSequence()` 를 사용할 수도 있음
- `generateSequence()`
  - 이전의 원소를 인자로 받아 다음 원소를 계산

## 문제

### 문제 1

```kotlin
data class Post(
    val id: Int,
    val tags: List<String>,
    val paragraph: String
)
```

- List<Post> 를 받아 태그별로 해당 문단을 길이별로 내림차순으로 정렬한 Map<String, List<String>> 을 반환하는 함수를 작성하시오.
  - 입력이 비어있을 때는 빈 맵을 반환.
  - 한 문단은 여러 태그를 가질 수 있음.

### 문제 2

```kotlin
data class User(val id: Int, val nickname: String)

val users = listOf(
    User(1, "fox"), User(2, "cat"), User(3, "fox")
)
val byNick = users.associateBy { it.nickname }
```

- 위 코드에서 byNick.size 의 값은?
  - 왜 그럴까요~
- 동일한 닉네임을 모두 보존하려면 어떻게 수정해야 할까요?
