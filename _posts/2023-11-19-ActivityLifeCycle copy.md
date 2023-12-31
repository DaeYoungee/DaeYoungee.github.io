---
title: "[Kotlin] Activity LifeCycle"
excerpt: "안드로이드 액티비티 라이프사이클에 대해 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin]
sidebar:
  nav: "counts"

data: 2023-11-19
last_modified_at: 2023-11-19

published: true
---

## Activity LifeCycle

**액티비티가 생성되고 소멸되기가지의 생명주기**이다. 다른 액티비티에 의해 가려진다던가 갑자기 전화가 걸려오는 등 모든 상황에 있어 **액티비티의 상태가 변화**한다.

안드로이드에서는 액티비티의 '상태가 변화' 할 때마다 특정 동작을 수행할 수 있도록 여러 콜백 메소드를 제공한다. 이를 'Lifecycle Callback Method' 라고 한다.

## Activity LifeCycle 동작 순서

콜백 메서드를 중심으로 나타낸 다이어그램이다.
`onCreate()` -> `onStart()` -> `onResume()` -> `onPause` -> `onStrop()` -> `onDestroy()` 의 콜백 메서드로 구성되어 있으며, 액티비티의 상태가 변경될 때마다 해당되는 콜백 메서드를 호출한다.

<div align="center">
  <img alt="액티비티 라이프사이클" src="https://github.com/DaeYoungee/DaeYoungee.github.io/assets/121485300/46e353b8-2660-4550-92e0-f94a66f67050">   
</div>

> 모든 안드로이드 콜백 메서드를 구성할 필요는 없고 그 떄마다 필요한 메서드를 구현하면된다. `onCreate()` 콜백 메서드는 무조건 오버라이딩해야하는 메서드이다.

## Activity LifeCycle Callback Method

### 1. onCreate()

시스템이 액티비티를 생성할 때 실행되는 콜백 메서드.
다른 콜백 메서드와는 다르게 `onCreate()` <span style="color:rgb(77,171,254)"> **은 무조건 오버라이딩하여 구현** </span> 해야한다.  
이 메서드는 액티비티 전체 수명 주기 동안 **딱 한 번만 동작되는 초기화 및 시작 로직**을 실행할 수 있다.

> 💡 TIP  
> `onCreate()`는 `savedInstanceState` 라는 파라미터를 수신하는데, 이는 액티비티의 이전 상태가 저장된 `Bundle` 객체이다.(처음 생성된 액티비티인 경우 null을 담고 있다.) 이를 통해 이전 상태를 복원하여 화면에 표시한다.

### 2. onStart()

액티비티가 `onCreate()`를 호출한 뒤 액티비티가 **STARTED** 상태에 진입하게 되면 `onStart()`가 호출된다. `onStart()`는 매우 바른 속도로 실행되고 `onResume()`메서드를 호출하게 된다.

### 3. onResume()

액티비티가 **RESUMED** 상태에 진입하게 되면 **포그라운드에 액티비티가 표시되고 앱이 사용자와 상호작용을 할 수 있는 상태**가 된다. 이떤 방해 이벤트 및 인터럽트(ex 전화)가 발생하여 사용자의 포커스가 없어지지 않는 이상, 이제 앱은 **RESUMED** 상태에 머무르게 된다.

> 💡 TIP  
> 만약 위에서 설명한 방해 이벤트 및 인터럽트가 발생하게 되면 어떤 일이 일어날까? -> 액티비티는 **PAUSED** 상태에 들어가게 되어 시스템이 onPause() 콜백 메서드를 호출하게 된다.

전화를 다 받고 오는 등, 액티비티가 다시 화면에 보여지면서 **RESUMED** 상태로 돌아오게 되면 시스템이 다시 한번 `onResume()`메서드를 호출하게 된다. 따라서, `onResume()`을 구현하여 `onPause()` 중에 해제되는 구성요소를 다시 초기화하고, 액티비티가 재개될 때마다 필요한 이외 초기화 작업을 수행하면 된다.

❗️ 어떤 액티비티 상태에서 초기화 작업을 실행하든간에 각각에 상응하는 수명 주기 이벤트를 사용해서 리소스를 해제해야한다. **STARTED** 이후에 무언가를 초기화 했다면, **STOPPPED** 이후에 이를 해제하거나 종료하면되고, **RESUMED** 이후에 무언가를 초기화 했다면, **PAUSED** 이후에 해제해주면 된다.

### 4. onPause()

A 액티비티에서 B 액티비티로 이동할 때(다른 액티비티에 포커스를 뒀을 때) A 액티비티에서 호출되는 콜백 메서드이다. 즉, 해당 **액티비티가 포그라운드에 있지 않게 되었다**는 것을 의미한다.

따라서 보통 `onPause()`에서는 액티비티가 포그라운드에 없을 동안 계속 실행되어서는 안 되지만 언젠가 다시 시작할 작업을 일시중지하는 작업을 수행

> 액티비티가 **PAUSED** 상태에 진입하게 되는 여러가지 루트들
>
> - 앱이 실행되던 중 방해 이벤트 및 인터럽트가 발생한 경우 (ex 전화 올 경우)
> - 화면에 반투명(?) 액티비티가 열리는 경우 (ex 권한 요청 다이얼로그)

또한, `onPause()` 메소드를 이용하여 배터리 수명에 영향을 미칠 수 있는 모든 시스템 리소스, 하드웨어 센서 등을 할당 해제하면 자원을 효율적으로 사용할 수 있다.

> ⚠️ 꼭 알아두자  
> `onPause()` 메서드는 아주 잠깐 실행되기 때문에 무언갈 저장하는 작업 실행하기엔 시간이 부족하다, `onPause()` 내에서는 사용자 데이터 저장, 네트워크 호출, DB 트랜잭션 등을 실행해서느 안 된다. 이렇게 부하가 큰 작업들은 `onStop()`에서 수행해야 한다.

### 5. onStop()

액티비티가 사용자에게 더 이상 표시되지 않으면 **STOPPED** 상태에 진입하고 시스템이 `onStop` 콜백 메서드를 호출한다. **새로 시작된 액티비티가 화면 전체를 차지할 경우**(그 전에 실행중인 앱이 백그라운드에서 돌아갈 경우)에 해당된다.

이 메소드에서는 필요하지 않은 리소스를 해제하거나 조정해야 한다. ex) 애니메이션을 일시 중지, GPS 사용 시 배터리를 아끼기 위해 위치 인식 정확도를 '세밀한 위치'에서 '대략적인 위치'로 전환할 수 있다.

> 💡 TIP  
> 액티비티가 **STOPPED** 상태에 들어가더라도 액**티비티 객체는 메모리 안에 머무른다**. 그런데 시스템이 더 우선순위가 높은 프로세스를 위해 **메모리를 확보**해야하는 경우, 이 **액티비티를 메모리 상에서 죽이게 된다.** 그럼에도 불구하고 **Bundle**에 View 객체 상태를 그대로 저장하고 사용자가 이 액티비티로 다시 돌아오게 되면 이를 기반으로 상태를 복원한다.

만약 사용자가 다시 액티비티로 돌아오게 되면 **STOPPED**상태에서 시작해 `onRestart()` -> `onStart()` -> `onResume()`을 연달아 호출하여 **RESUMED** 상태로 변화하게 된다. 액티비티가 포그라운드로 올라오면서 사용자와 상호작용을 다시 시작한다.

### 6. onDestroy()

액티비티가 **완전히 소멸되기 전에** 이 콜백 메서드가 호출된다.

1. `finish()` 가 호출되거나 **사용자가 앱을 종료**하여 액티비티가 종료되는 경우
2. **화면 구성(configuration)이 변경**되어 일시적으로 액티비티를 소멸시키는 경우, ex) 기기 회전

> ⚠️ 꼭 알아두자  
> 만액 `onDestroy()` 가 호출되기 까지 **해제되지 않은 리소스가 있다**면, 모두 여기서 **해제**해줘야 한다. **Memory Leak**(메모리 누수)의 위험이 있다.

## Reference

https://velog.io/@haero_kim/Activity-Lifecycle-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0
