---
title: "[Kotlin] Koltin 스톱워치"
excerpt: "Koltin에서 시간이 증가하는 기능을 구현해보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, StopWatch]
sidebar:
  nav: "counts"

data: 2024-01-08
last_modified_at: 2024-01-08

published: true
---

## 스톱워치

이번 포스팅에서는 코틀린에서 스톱워치 기능을 구현하는 2가지 방법을 소개한다.

1. **LocalDateTime을 이용하는 방법** -> 오류 발생
2. **TimeUnit을 이용하는 방법** -> 훨씬 간결하다, 추천

<iframe src="https://github.com/Google-Developer-Student-Clubs-TUK/2024-Solution-Challenge-Team-WaterSavior/assets/121485300/9ecb01fd-09d6-40a0-a6e2-ede5726d6470" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<br>

## currentTimeMillis()

`System.currentTimeMillis()`을 이용하면 현재 시간을 Long타입으로 변환해서 알려준다.

```kotlin
viewModelScope.launch {
    lastTimestamp = System.currentTimeMillis()          // 1초 전 시각
    delay(1000)
    timeMillis += System.currentTimeMillis() - lastTimestamp // 현재 시각
}
```

현재 시각 - 1초 전 시각 = 1초 가 나온다.  
1초를 계속 더하면 StopWatch 기능을 구현할 수 있다고 생각했다!!
위와 같은 방법에는 기가막힌 오류가 있었다. 오류는 나중에 살펴보고 간단한 코드를 설명해보겠다.

- 전체 코드

  ```kotlin
    @RequiresApi(Build.VERSION_CODES.O)
    fun startTime() {
        job3 = viewModelScope.launch {
            lastTimestamp = System.currentTimeMillis()
            while (recoding.value) {
                delay(1000)
                timeMillis += System.currentTimeMillis() - lastTimestamp
                Log.d("daeYoung", "currentTimeMillis: ${formatTime(System.currentTimeMillis())}")
                Log.d("daeYoung", "lastTimestamp: ${lastTimestamp}")
                lastTimestamp = System.currentTimeMillis()
                _formattedTime.value = formatTime(timeMillis)
                Log.d("daeYoung", "time: ${_formattedTime.value}")
            }
        }
    }

    @RequiresApi(Build.VERSION_CODES.O)
    private fun formatTime(timeMillis: Long): String {
    //        val localDateTime = LocalDateTime.ofInstant(
    //            Instant.ofEpochMilli(timeMillis),
    //            ZoneId.systemDefault()
    //        )
        val localDateTime = Instant.ofEpochMilli(timeMillis).atZone(ZoneId.systemDefault()).toLocalDateTime()
        val formatter = DateTimeFormatter.ofPattern(
            "HH : mm : ss",
            Locale.getDefault()
        )
        return localDateTime.format(formatter)
    }
  ```

  > LocalDate, LocalDateTime 클래스는 Android 버전 코드 오레오(Build.VERSION_CODES.O) / API level 26 이상부터 지원이 됩니다.

  당연히 우리의 어플은 minSdk가 26 미만일 것이다. 그래서, 하위 버전에서도 위의 클래스를 지원하도록 해줘야 한다. 그래서 `@RequiresApi(Build.VERSION_CODES.O)`을 추가한다.  
  <br>
  만약 `@RequiresApi(Build.VERSION_CODES.O)`을 추가하는 방법이 귀찮다면 gradle(모듈수준)에 dependency를 추가해주면 된다.

  ```kotlin
  android {
    ...
    compileOptions {
        coreLibraryDesugaringEnabled true   // LocalDateTime 이 Api 26 이하 지원을 위해서 추가함.
        ...
    }
  }

  dependencies {
      coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.1.5'
      ...
  }
  ```

## milliseconds -> LocalDateTime

milliseconds를 원하는 format으로 변환하기 위해서는 먼저 LocalDateTime을 바꿔줘야한다.  
ex) milliseconds를 ""HH : mm : ss : SSS"로 바꿀 때 사용

```kotlin
private var timeMillis = 0L
//        val localDateTime = LocalDateTime.ofInstant(
//            Instant.ofEpochMilli(timeMillis),
//            ZoneId.systemDefault()
//        )
        val localDateTime = Instant.ofEpochMilli(timeMillis).atZone(ZoneId.systemDefault()).toLocalDateTime()
```

localDateTime으로 바꿔주는 코드는 2개가 있고(주석과, 주석아래의 코드) 입맛에 맞는것으로 골라 사용하면된다.

## LocalDateTime -> 원하는 포멧

LocalDateTime을 ""HH : mm : ss"로 바꿔보자.

```kotlin
val formatter = DateTimeFormatter.ofPattern(
    "HH : mm : ss",
    Locale.getDefault()
)
return localDateTime.format(formatter)
```

## StopWatch 코드 구현(문제 발생)

**현재시각 - 1초 전 시각 = 1초** 을 활용해서 1초마다 증가시키는 코드를 만들어 StopWatch를 구현하자.

```kotlin
private var timeMillis = 0L       // 현재시각(밀리초)
private var lastTimestamp = 0L    // 1초전 시각(밀리초)

fun startTime1() {
    job3 = viewModelScope.launch {
        lastTimestamp = System.currentTimeMillis()
        while (true) {   // 계속 1초씩 증가하는 무한루프
            delay(1000)
            timeMillis += System.currentTimeMillis() - lastTimestamp
            lastTimestamp = System.currentTimeMillis()
            _formattedTime.value = formatTime1(timeMillis)
        }
    }
}

private fun formatTime1(timeMillis: Long): String {
    val localDateTime = Instant.ofEpochMilli(timeMillis).atZone(ZoneId.systemDefault()).toLocalDateTime()
    val formatter = DateTimeFormatter.ofPattern(
        "HH : mm : ss",
        Locale.getDefault()
    )
    return localDateTime.format(formatter)
}
```

위의 코드만 봤을 때 문제 없이 잘 돌아가야한다. **그러나 문제가 생겼다.**  
**현재시각 - 1초 전 시각** 을 "HH : mm : ss"로 바꿨을 때 "00 : 00 : 01"로 시작해야한다. 그러나 아래의 영상과 같이 "09 : 00 : 01" 로 9시부터 보여진다.

<iframe src="https://github.com/DaeYoungee/Compose_study/assets/121485300/1760746e-f7e6-494c-b321-0532acf05895" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" muted=1 allowfullscreen></iframe>

<br>

## TimeUnit 이용(해결법)

**TimeUnit**을 이용한 코드는 위의 방법보다 훨씬 간단하다.

- `TimeUnit.MILLISECONDS.toHours(milliseconds) % 24` 밀리초를 시간으로 바꿔주고 24보다 숫자가 작게 나온다.
- `TimeUnit.MILLISECONDS.toMinutes(milliseconds) % 60` 밀리초를 분으로 바꿔주고 60보다 숫자가 작게 나온다.
- `TimeUnit.MILLISECONDS.toSeconds(milliseconds) % 60` 밀리초를 초로 바꿔주고 60보다 숫자가 작게 나온다.

```kotlin
 // 시간 측정
    @RequiresApi(Build.VERSION_CODES.O)
    fun startTime() {
        job3 = viewModelScope.launch {
            while (true) {
                delay(1000)
                milliseconds += 1000L
                _formattedTime.value = formatTime(milliseconds)
            }
        }
    }

    // 시간 format 설정
    @RequiresApi(Build.VERSION_CODES.O)
    private fun formatTime(milliseconds: Long): String {
        val hours = TimeUnit.MILLISECONDS.toHours(milliseconds) % 24
        val minutes = TimeUnit.MILLISECONDS.toMinutes(milliseconds) % 60
        val seconds = TimeUnit.MILLISECONDS.toSeconds(milliseconds) % 60
        val formatter = String.format("%02d : %02d : %02d", hours, minutes, seconds)
        return formatter
    }
```

위의 코드로 실행하면 처음에 본 영상과 같이 성공적으로 stopwatch 기능을 구현할 수 있다.

## References

[milliseconds -> LocalDateTime](https://heeeju4lov.tistory.com/36)  
[LocalDateTime -> 원하는 포멧](https://www.baeldung.com/java-convert-epoch-localdate)  
[Compose를 활용하여 스탑워치 데스크탑앱 만들기](https://velog.io/@blucky8649/%EC%BD%94%ED%8B%80%EB%A6%B0-Compose%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%98%EC%97%AC-%EB%8D%B0%EC%8A%A4%ED%81%AC%ED%83%91-%EC%8A%A4%ED%83%91%EC%9B%8C%EC%B9%98-%EB%A7%8C%EB%93%A4%EA%B8%B0)
[TimeUnit 이용](https://www.programiz.com/kotlin-programming/examples/milliseconds-minutes-seconds)
