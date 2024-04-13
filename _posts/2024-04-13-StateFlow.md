---
title: "[Kotlin] StateFlow "
excerpt: "kotlin에서 HotStream인 StateFlow에 대해 알아보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Flow]
sidebar:
  nav: "counts"

data: 2024-04-13
last_modified_at: 2024-04-13

published: true
---

**StateFlow**는 **SharedFlow**를 확장시킨 것으로, replay 인자 값이 1인 **SharedFlow**와 비슷하게 작동한다.  
**StateFlow**는 _value_ 프로퍼티로 접근 가능한 값 하나를 항상 갖고 있다.

### MutableStateFlow의 상속

**MutableStateFlow**는 **MutableSharedFlow**와 **StateFlow**를 구현하고 있다.  
**MutableStateFlow**는 일반적인 **State**와 동일하게 **value** 프로퍼티를 통해 값에 접근이 가능하다.

![image](https://github.com/DaeYoungee/Compose_study/assets/121485300/cbf471a5-1c8c-49f1-ba02-d80ae4454462)

## MutableStateFlow

**MutableStateFlow**는 value 프로퍼티로 값을 얻어올 수도 있고 설정할 수도 있습니다.  
**MutableStateFlow**은 값은 감지할 수 있는 보관소이다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val state = MutableStateFlow("A")
    println(state.value)  // A

    launch {
        state.collect { println("Value changed to $it") }
        // Value changed to A
    }

    delay(1000)
    state.value = "B"   // Value changed to B

    delay(1000)

    launch {
        state.collect { println("and not it is $it") }
        // and not it is B
    }

    delay(1000)
    state.value = "C"
    // Value changed to C
    // and not it is C
}
```

<br>

안드로이드에서 **MutableStateFlow**는 LiveData를 대체하는 최신 방식으로 사용되고 있다.
**MutableStateFlow**는 초기값을 갖고 있기 때문에 null일 필요가 없다.  
**MutableStateFlow**는 ViewModel에서 상태를 나타낼 때 주로 사용된다. **MutableStateFlow**의 상태를 감지할 수 있으며, 감지된 상태에 따라 뷰가 보여지고 갱신된다.

### MutableStateFlow는 하나의 값을 저장

**MutableStateFlow**는 데이터가 덮어 씌어지기 때문에, 관찰이 느린 경우 상태의 중간변화를 받을 수 없는 경우도 있다. **모든 이벤트를 다 받으려면** <span style="color:rgb(77,171,254)">**SharedFlow**</span>**를 사용해야 한다.**

```kotlin
suspend fun main(): Unit = coroutineScope {
    val state = MutableStateFlow('X')

    launch {
        for (c in 'A'..'E') {
            delay(300)
            state.value = c
        }
    }
    state.collect {
        delay(1000)
        println(it)
    }
    // X
    // C
    // E
}
```

### stateIn

**stateIn**는 **Flow**를 **StateFlow**로 변환하는 함수이다.  
<span style="color:rgb(77,171,254)">**MutableStateFlow는 항상 값을 가져야 한다.**</span> 따라서 값을 명시하지 않았을 때는 첫 번째 값이 계산될 때까지 기다려야 한다.

아래의 예시에서 `flow.stateIn(this)` 하고 바로 **MutableStateFlow**에 "A"를 저장하기 위해 `delay(1000)`과 `println("Produced $it")`를 순차적으로 실행한다.  
이 후 **MutableStateFlow**에 "A" 값이 저장되고 `println("Listening")`부터 순차적으로 코드가 실행된다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val flow = flowOf("A", "B", "C")
        .onEach { delay(1000) }
        .onEach { println("Produced $it") }
    val stateFlow = flow.stateIn(this)

    println("Listening")
    println(stateFlow.value)
    stateFlow.collect { println("Received $it") }

    // (1초 후)
    // Produced A
    // Listening
    // A
    // Received A
    // (1초 후)
    // Produced B
    // Received B
    // (1초 후)
    // Produced C
    // Received C
}

```

<br>

다음으로 `stateIn()`의 두번 째 형태를 살펴보자. initialValue(초기값)과 started 모드를 설정해야 한다. started 모드의 설정 갑은 SharedFlow의 started 모드 설정 값과 동일하다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val flow = flowOf("A", "B", "C")
        .onEach { delay(1000) }
        .onEach { println("Produced $it") }
    val stateFlow = flow.stateIn(
        scope = this,
        started = SharingStarted.Lazily,
        initialValue = "Empty"
    )

    println(stateFlow.value)

    delay(2000)
    stateFlow.collect { println("Received $it") }

    // Empty
    // (2초 후)
    // Received Empty
    // (1초 후)
    // Produced A
    // Received A
    // (1초 후)
    // Produced B
    // Received B
}
```

<br>

> 💡 <span style="font-size:15pt;">StateFlow의 **value 프로퍼티**와 **collect{} 메서드**의 차이</span>
>
> `StateFlow의.value`는 StateFlow에 현재 저장되어 있는 값 하나를 읽고 끝난다.  
>  반면에 `StateFlow의.collect{}`는 죽지 않고 계속해서 StateFlow의 변화를 감지하며 데이터가 변경될 때마다 `collect{}` 문을 실행한다.
