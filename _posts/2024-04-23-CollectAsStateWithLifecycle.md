---
title: "[Kotlin Compose] CollectAsState와 CollectAsStateWithLifecycle "
excerpt: "Compose에서 CollectAsState의 사용과 CollectAsStateWithLifecycle 필요성에 대해 알아보자. "

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose]
sidebar:
  nav: "counts"

data: 2024-04-23
last_modified_at: 2024-04-23

published: true
---

Jetpack Compose로 Android 앱을 빌드하는 경우 collectAsStateWithLifecycleAPI를 사용하여 UI에서 수명 주기 인식 방식으로 흐름을 수집해야 한다.  
`collectAsStateWithLifecycle`를 사용하면 앱이 백그라운드에 있어 사용하지 않을 때 불필요한 리소스 낭비를 방지할 수 있다.

## collectAsState

Ui 에서 mutableStateFlow.value만 사용해서는 값이 변경되었을 때 리컴포지션이 일어나지 않는다. `collectAsState`을 통해서 flow를 state로 변환해주고 State.value를 Ui 에서 사용하면 값이 변경되었을 때 리컴포지션이 일어난다.

`collectAsState` 는 컴포지션의 수명 주기를 따릅니다. 컴포저블이 컴포지션에 들어갈 때 흐름 수집을 시작하고 컴포지션을 떠날 때 수집을 중지합니다.  
Android 앱이 백그라운드에 있는 동안 Compose가 재구성을 중단하더라도 **collectAsState 컬렉션을 활성 상태로 유지**합니다. 이로 인해 나머지 계층 구조에서 리소스를 확보할 수 없습니다.

`collectAsState` 는 다른 플랫폼용으로 개발할 때 사용, `collectAsStateWithLifecycle` 은 Android 앱을 개발할 때 사용한다.

## CollectAsStateWithLifecycle

`collectAsStateWithLifecycle` 흐름에서 값을 수집하고 State 수명 주기 인식 방식으로 Compose로 최신 값을 나타내는 Composable 함수이다. 새로운 flow 방출이 발생할 때마다 이 State객체의 값이 업데이트된다. 이로 인해 State.value를 사용한 컴포저블 함수는 리컴포지션된다.

<br>

![image](https://github.com/DaeYoungee/Compose_study/assets/121485300/ada831ab-d658-499e-bef9-23ba7e1c9304)

`collectAsStateWithLifecycle`는 minActiveState 매개변수를 통해 flow의 값을 수집하는 생명 주기를 정할 수 있다. `Lifecycle.State.STARTED`는 디폴트 값으로 Activity가 시작되어 **onStart**가 호출된 직후부터 **onPause**가 호출되기 직전까지 flow의 수집을 가능하게 한다는 뜻이다.

<br>

<img width="403" alt="image" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/bd7c7d84-18d7-4c9e-b5fa-024aa905220b">

## 종속 항목 추가

`CollectAsStateWithLifecycle`를 사용하기 위해서는 module 수준의 gradle파일에 종속 항목을 추가해줘야 한다.

```kotlin
dependencies {
    implementation "androidx.lifecycle:lifecycle-runtime-compose:2.6.0"
}
```

## Reference

[medium](https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3)
