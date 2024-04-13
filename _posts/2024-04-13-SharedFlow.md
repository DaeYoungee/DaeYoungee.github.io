---
title: "[Kotlin] SharedFlow "
excerpt: "kotlin에서 HotStream인 SharedFlow에 대해 알아보자."

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

일반적인 **ColdStream**인 **Flow**는 소비자가 요청할 때마다 데이터를 생산하고 계산한다. 이와는 다르게 소비자가 데이터를 요청하지 않아도 데이터를 생산하고 계산하는 **HotStream**에 대해 알아보자.

먼저 **HotStream**의 3가지 특징을 살펴보자.

1. 하나의 생산자에 여러 소비자들이 값의 변화를 확인할 수 있다.(브로드 캐스트)
2. flow 빌더 내부가 아닌 밖에서도 데이터 생산이 가능하다.
3. 소비자가 데이터를 요청하지 않아도 생산자가 데이터를 생산한다.

## SharedFlow

SharedFlow를 통해 메시지를 보내면 대기하고 있는 모든 코루틴이 수신하게 된다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val mutableSharedFlow = MutableSharedFlow<String>(replay = 0)

    launch {
        mutableSharedFlow.collect {
            println("#1 received $it")
        }
    }

    launch {
        mutableSharedFlow.collect {
            println("#2 received $it")
        }
    }

    delay(1000)
    mutableSharedFlow.emit("Message1")   // 외부에서 데이터 생산 가능
    mutableSharedFlow.emit("Message2")
}
// #1 received Message1
// #2 received Message1
// #2 received Message2
// #1 received Message2
```

> 💡 위 프로그램은 coroutineScope의 자식 코루틴이 launch로 시작된 뒤 MutableSharedFlow를 갑지하고 있는 상태이므로 **종료되지 않는다.** 물론 MutableSharedFlow를 종료할 방법은 없으므로 프로그램을 종료하려면 전체 스코프를 취소하는 방법밖에 없다.

### replay

MutableSharedFlow는 일반적으로 데이터를 생산하고 바로 수집하지 않으면 데이터를 소멸된다. 이러한 무의미한 행동을 방지하기 위해 **replay**를 사용한다. **replay** 인자를 설정하면 마지막으로 전송한 값들이 정해진 수만큼 저장된다.

**replayCache**는 cash에 저장되어있는 데이터를 List 타입으로 반환해준다.  
**resetReplayCache**는 cash에 저장되어있는 데이터를 초기화시킨다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val mutableSharedFlow = MutableSharedFlow<String>(replay = 2)

    mutableSharedFlow.emit("Message1")
    mutableSharedFlow.emit("Message2")
    mutableSharedFlow.emit("Message3")

    println(mutableSharedFlow.replayCache)
    // [Message2, Message3]

    launch {
        mutableSharedFlow.collect {
            println("#1 received $it")
            // #1 received Message2
            // #1 received Message3
        }
    }

    delay(100)
    mutableSharedFlow.resetReplayCache()
    println(mutableSharedFlow.replayCache)
    // []
}
```

### MutableSharedFlow 내부

**MutableSharedFlow**는 **SharedFlow**를 구현하고 있고 감지 목적으로 사용되며, **FlowCollector**는 값을 방출하는 목적으로 사용된다.

- MutableSharedFlow  
  <img width="459" alt="image" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/ecb7abaf-f90e-4bb0-933a-8140606d15df">

<br>

- FlowCollector  
  <img width="663" alt="image" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/54678509-274e-47e5-a1e6-b45e45cc0a2a">

<br>

- SharedFlow  
  **SharedFlow**는 **Flow**를 구현하고 있다.
  <img width="676" alt="Screenshot 2024-04-13 at 3 28 58 PM" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/29fb7ccf-fe5d-4c79-a342-610ea9a15ece">

### shareIn

**shareIn**은 **Flow**를 **SharedFlow**로 바꾸는 함수이다.

```kotlin
suspend fun main(): Unit = coroutineScope {
    val flow = flowOf("A", "B", "C").onEach { delay(1000) }

    val sharedFlow: SharedFlow<String> = flow.shareIn(
        scope = this,
        started = SharingStarted.Eagerly,
        // replay = 0 (default)
    )

    delay(500)

    launch {
        sharedFlow.collect { println("#1 $it") }
    }

    delay(1000)

    launch {
        sharedFlow.collect { println("#2 $it") }
    }

    delay(1000)

    launch {
        sharedFlow.collect { println("#3 $it") }
    }

    // (1초 후)
    // #1 A
    // (1초 후)
    // #1 B
    // #2 B
    // (1초 후)
    // #1 C
    // #2 C
    // #3 C
}
```

**shareIn** 함수는 SharedFlow를 만들고 Flow의 원소를 보낸다. 플로우의 원소를 모으는 코루틴을 시작하므로 **sharedIn** 함수의 첫 번째 인자로 코루틴 스코프를 받는다. 세번째 인자는 **replay**로 default 값은 0이다.  
여기서 주의깊게 바라볼 점은 두 번째 인자인 **started**이다. **started**은 리스너의 수에 따라 값을 언제부터 감지할지 결정한다.

- SharingStarted.Eagerly  
   값을 즉시 감지하고 플로우로 값을 전송한다. 값을 감지하지 전에 방출을 하면 값의 일부를 유실할 수 있다.  
   (아래의 코드에서는 replay가 0이므로 먼저 방출한 값 전부 유실된다.)

  ```kotlin
    suspend fun main(): Unit = coroutineScope {
        val flow = flowOf("A", "B", "C")
        val sharedFlow: SharedFlow<String> = flow.shareIn(
        scope = this,
        started = SharingStarted.Eagerly,
        // replay = 0 (default)
        )

        delay(100)

        launch {
        sharedFlow.collect { println("#1 $it") }
        }
        println("Done")

        // (0.1초 후)
        // Done
    }
  ```

  <br>

- SharingStarted.Lazily
  첫 번째 구독자가 나올 때 감지를 시작한다. 첫번 째 구독자는 내보내진 모든 값을 수신하는 것이 보장되며, 이후의 구독자는 replay 수만큼 가장 최근에 저장된 값들을 받게 된다. 모든 구독자가 사라져도 업스트림(데이터를 방출하는) 플로우는 액티브 상태지만, 구독자가 없으면 replay 수만큼 가장 최근의 값들만 캐싱한다.

  ```kotlin
  suspend fun main(): Unit = coroutineScope {
      val flow1 = flowOf("A", "B", "C")
      val flow2 = flowOf("D").onEach { delay(1000) }

      val sharedFlow = merge(flow1, flow2).shareIn(
          scope = this,
          started = SharingStarted.Lazily,
      )

      delay(100)
      launch {
          sharedFlow.collect { println("#1 $it") }
      }
      delay(1000)
      launch {
          sharedFlow.collect { println("#2 $it") }
      }

      // (0.1 초 후)
      // #1 A
      // #1 B
      // #1 C
      // (1 초 후)
      // #2 D
      // #1 D
  }
  ```

  <br>

- SharingStarted.WhileSubscribed  
   첫 번째 구독자가 나오면 감지를 시작하며 마지막 구독자가 사라지면 플로우도 멈춘다. SharedFlow가 멈췄을 때 새로운 구독자가 나오면 플로우가 다시 시작된다.  
   **WhileSubscribed**는 (기본값이 0이며, 마지막 구독자가 사라지고 난 뒤 감지할 시간을 나타내는)**stopTimeOutMillis**와 (기본값은 Long.MAX_VALUE이며 멈춘 뒤 replay 값을 가지고 있는 시간을 나타내는) **replayExpirationMilles**라는 설정 파라미터를 추가로 갖고 있다.

  ```kotlin
  suspend fun main(): Unit = coroutineScope {
    val flow = flowOf("A", "B", "C", "D")
        .onStart { println("Started") }
        .onCompletion { println("Finished") }
        .onEach { delay(1000) }

    val sharedFlow = flow.shareIn(
        scope = this,
        started = SharingStarted.WhileSubscribed(),
    )


    delay(3000)
    launch {
        println("#1 ${sharedFlow.first()}")
    }
    launch {
        println("#2 ${sharedFlow.take(2).toList()}")
    }
    delay(3000)
    launch {
        println("#3 ${sharedFlow.first()}")
    }

    // (3초 후)
    // Started
    // (1초 후)
    // #1 A
    // (1초 후)
    // #2 [A, B]
    // Finished
    // (1초 후)
    // Started
    // (1초 후)
    // #3 A
    // Finished
  }
  ```
