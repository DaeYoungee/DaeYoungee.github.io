---
title: "[Kotlin]Flow Intermediary(중간연산자) "
excerpt: "Kotlin Flow에서 제공하는 중간연산자에 대해 알아보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Flow, Intermediary]
sidebar:
  nav: "counts"

data: 2024-04-17
last_modified_at: 2024-04-17

published: true
---

flow는 **순차적으로 값을 배출**하는 **비동기 데이터 스트림**이다. 플로우에서는 값이 순차적으로 흐르는데 플로우 생성과 최종 연산 사이 중간연산이 가능하다. 이러한 연산들을 **플로우 처리**(flow processing)이라고 한다.

### map

가장 먼저 배울 중요한 함수는 flow의 값을 입맛에 맞게 변형하는 map 함수이다.  
`map()`은 데이터를 순차적으로 변형하고 새로운 flow를 반환한다.

<img width="312" alt="image" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/5a91e00e-e937-4e58-9390-dff3a49cbbd8">

<br>

```kotlin
suspend fun main(): Unit = coroutineScope {
    flowOf(1, 2, 3)                   // [1, 2, 3]
        .map { it * it }              // [1, 4, 9]
        .collect { print(it) }        // 149
}
```

### filter

filter 또한 중요한 함수이며, 원래 플로우에서 **주어진 조건에 맞는 값들만 가진 플로우**로 반환한다.  
**filter는 관심 없는 원소를 제거할 때 주로 사용된다.**

```kotlin
suspend fun main(): Unit = coroutineScope {
    (1..10).asFlow()                     // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
        .filter { it <= 5 }              // [1, 2, 3, 4, 5]
        .filter { it % 2 == 0 }          // [2, 4]
        .collect { print(it) }           // 24
}
```

### take

**특정 수의 원소만 통과**시키기 위해 take를 사용한다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    ('A'..'Z').asFlow()                 // [A, B, C, .. , Z]
        .take(5)                        // [A, B, C, D, E]
        .collect { print(it) }          // ABCDE
}
```

### drop

**특정 수의 원소를 무시**시키기 위해 drop을 사용한다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    ('A'..'Z').asFlow()                // [A, B, C, .. , Z]
        .drop(20)                      // [U, V, W, X, Y, Z]
        .collect { print(it) }         // UVWXYZ
}
```

## flow 결합

여러개의 flow를 합칠 때 3가지의 메서드가 있다. `merge`, `zip`, `combine`

### merge

**두개의 flow를 하나로 합칠 때** merge를 사용한다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val ints: Flow<Int> = flowOf(1, 2, 3)
    val doubles: Flow<Double> = flowOf(0.1, 0.2, 0.3)

    val together: Flow<Number> = merge(ints, doubles)
    print(together.toList())

    // [1, 0.1, 0.2, 0.3, 2, 3]
    // 또는 [1, 0.1, 2, 3, 2, 0.2, 0.3]
    // 또는 [1, 2, 3, 0.1, 0.2, 0.3]
    // 또는 [0.1, 1, 0.2, 0.3, 2, 3]
}
```

`merge()`를 사용하면 한 플로우의 원소가 다른 플로우를 기다리지 않는다는 것이 중요하다. 예를 들어 첫 번째 플로우의 원소 생성이 지연된다고 해서 두 번째 플로우의 원소 생성이 중단되지 않는다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val ints: Flow<Int> = flowOf(1, 2, 3).onEach { delay(1000) }
    val doubles: Flow<Double> = flowOf(0.1, 0.2, 0.3)

    val together: Flow<Number> = merge(ints, doubles)
    together.collect { print(it) }

    // 0.1
    // 0.2
    // 0.3
    // (1초 후)
    // 1
    // (1초 후)
    // 2
    // (1초 후)
    // 3
}
```

실제 사용에서는 `merge()`는 여러 개의 이벤트들을 똑같은 방법으로 처리할 떄 사용한다.

###  zip

...

## FlatMap

컬렉션에서 잘 알려진 도 다른 함수는 flatMap이다. 컬렉션의 경우 flatMap은 map과 비슷하지만 변환 함수가 평탄화된 컬렉션을 반환한다는 점이 다르다.  
`map()`은 flow내의 원소를 내 입맛대로 변형하여 flow로 반환한다면,  
`flatMap()`은 flow와 다른 flow를 결합하여 새로운 flow를 반환한다.

### FlatMapConcat

flatMapConcat은 평탄화 관련 중간 연산자에 있어 가장 기본이 되는 메서드이다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    flowOf("A","B","C")
        .flatMapConcat { productNewFlow(it) }
        .collect { println("flatMapConcatTest: $it") }

    /* 결과값
    1 + A
    2 + A
    3 + A
    1 + B
    2 + B
    3 + B
    1 + C
    2 + C
    3 + C
    */
}
fun productNewFlow(element: String) =
    flowOf(1,2,3)
        .onEach { delay(1000L) }
        .map { "$it + $element" }
```

<br>

flatMapConcat은 하위 데이터 스트림의 생산부 처리를 모두 끝내고 상위 데이터 스트림의 값을 차례대로 방출한다.

상위 데이터 스트림의 첫 번째 원소인 A를 방출 -> 하위 데이터 스트림 연산 -> 상위 데이터 스트림의 두 번째 원소인 B를 방출-> 하위 데이터 스트림 연산

이 와 같은 순서로 이뤄진다.

> <span style="font-size:13pt; color:rgb(235, 64, 52)">즉, FlatMapConcat은 동기적으로 스트림을 결합한다.</span>

### flatMapMerge

flatMapConcat은 하위 데이터 스트림의 생산부 처리와 상관없이 상위 데이터 스트림의 값을 차례대로 방출한다. 즉, 하위 데이터 스트림의 생산부 처리가 끝나지 않아도 상위 데이터 스트림의 값은 계속해서 방출된다는 뜻이다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    flowOf("A", "B", "C")
        .flatMapMerge { productNewFlow(it) }
        .collect { println("flatMapMergeTest: $it") }

    /*결과값
    1 + A
    1 + B
    1 + C
    2 + A
    2 + B
    2 + C
    3 + A
    3 + C
    3 + B
    */

    // or
    /*
    1 + C
    1 + A
    1 + B
    2 + B
    2 + A
    2 + C
    3 + A
    3 + C
    3 + B
     */
}

fun productNewFlow(element: String) =
    flowOf(1, 2, 3)
        .onEach { delay(1000L) }
        .map { "$it + $element" }
```

<br>

상위 데이터 스트림의 원소인 A, B, C를 동시에 방출 -> 하위 데이터 스트림 연산(1초 delay) -> 하위 데이터 스트림 연산((1 + A), (1 + B), (1 + C)를 동시에 방출) -> 하위 데이터 스트림 연산(1초 delay) -> 하위 데이터 스트림 연산((2 + A), (2 + B), (2 + C)를 동시에 방출)

이 와 같은 순서로 이뤄진다.

> <span style="font-size:13pt; color:rgb(235, 64, 52)">즉, flatMapMerge는 비동기적으로 스트림을 결합한다.</span>

### flatMapLatest

flatMapLatest은 동기적인 스트림을 지원한다.  
flatMapLatest 함수는 새로운 플로우가 나타나면 이전에 처리하던 플로우를 잊어버린다.새로운 값이 나올 떄마다 이전 플로우 처리는 사라져 버린다. **flatMapLatest**는 **flatMapConcat**, **flatMapMerge** 와는쓰임새가 다르다.

> <span style="font-size:12pt; font-weight:bold;">[flatMapConcat과 flatMapMerge의 공통점]</span>  
> 상위 스트림의 데이터 생산 속도 > 하위 스트림의 데이터 생산 속도

> <span style="font-size:12pt; font-weight:bold;"> flatMapLatest</span>  
> 상위 스트림의 데이터 생산 속도 < 하위 스트림의 데이터 생산 속도

```kotlin
suspend fun main(): Unit = coroutineScope {
    flowOf("A","B","C")
        .onEach { delay(1200L) }
        .flatMapLatest { productNewFlow(it) }
        .collect { println("flatMapLatestTest: $it") }

    /*결과값
    1 + A
    1 + B
    1 + C
    2 + C
    3 + C
    */
}

fun productNewFlow(element: String) =
    flowOf(1, 2, 3)
        .onEach { delay(1000L) }
        .map { "$it + $element" }
```

<br>

상위 데이터 스트림 연산(1.2초 delay)  
-> 상위 데이터 스트림의 첫 번째 원소인 A를 방출  
-> 하위 데이터 스트림 연산(1초 delay)와 동시에 상위 데이터 스트림 연산(1.2초 delay)  
-> 하위 데이터 스트림 연산(1 + A를방출)  
-> 하위 데이터 스트림 연산(1초 delay), 상위 데이터 스트림 연산(1.2초 delay)마치고 두 번째 원소인 B 방출  
-> 하위 데이터 스트림 연산(0.8초 delay 멈춤)  
-> 하위 데이터 스트림 연산(1초 delay)와 동시에 상위 데이터 스트림 연산(1.2초 delay)  
-> 하위 데이터 스트림 연산(1 + B를방출)  
-> 하위 데이터 스트림 연산(1초 delay), 상위 데이터 스트림 연산(1.2초 delay)마치고 두 번째 원소인 C 방출
-> ...

이 와 같은 순서로 이뤄진다.

위 코드를 보면 알다시피, 상위 스트림의 데이터 생산속도가 더 길다. 상위 스트림의 데이터 생산 속도는 1.2초마다 생산이 되며, 하위 스트림의 데이터 생산 속도는 1초이다. flatMapLatest는 Latest라는 이름이 포함된 것 처럼 최신의 값만 방출하는 것을 알 수 있다.

> <span style="font-size:12pt; font-weight:bold;">flatMapLatest는 하위 스트림 데이터 처리 여부를 상관하지 않는다.</span>  
> 오로지 상위 스트림의 최신 데이터가 방출되면 기존에 진행되던 하위 스트림의 데이터 처리는 cancel시키고 상위 스트림에 맞는 데이터 처리를 진행한다.

## flow 중복 제거 함수

반복되는 원소가 동일하다고 판단되면 제거하면 **distinctUntilChanged** 함수도 아주 유용하다. 이 함수는 바로 이전의 원소와 동일한 원소만 제거한다.  
**distinctUntilChangedBy**는 두 원소가 동일한지 판단하기 위해 비교할 키(key)를 인자로 받는다.  
**distinctUntilChanged**는 람다 표현식을 받아 두 원소가 비교되는 방법을 정의한다.

```kotlin
data class User(
    val id: Int,
    val name: String
) {
    override fun toString(): String = "[$id] $name"
}
suspend fun main(): Unit = coroutineScope {
    val users = flowOf(
        User(1, "Alex"),
        User(1, "Bob"),
        User(2, "Bob"),
        User(2, "Celine"),
    )
    println(users.distinctUntilChangedBy { it.id }.toList())
    println(users.distinctUntilChangedBy { it.name }.toList())
    println(users.distinctUntilChanged { prev, next -> prev.id == next.id || prev.name == next.name }.toList())

    /*결과값
    [[1] Alex, [2] Bob]
    [[1] Alex, [1] Bob, [2] Celine]
    [[1] Alex, [2] Bob]
    */
}
```

## Reference

[flatMapConcat, flatMapMerge, flatMapLatest](https://velog.io/@squart300kg/flatMapConcat-flatMapMerge-flatMapLatest%EC%B0%A8%EC%9D%B4)
