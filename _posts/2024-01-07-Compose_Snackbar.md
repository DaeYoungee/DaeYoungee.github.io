---
title: "[Kotlin] Koltin Jetpack Compose Snackbar 컴포저블"
excerpt: "Koltin Jetpack Compose에서 Snackbar가 무엇인지, 커스텀하는 방법을 알아보자 "

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose, Snackbar]
sidebar:
  nav: "counts"

data: 2024-01-07
last_modified_at: 2024-01-07

published: true
---

## Snackbar란?

> Snackbar는 화면 하단의 메시지를 통해 작업에 대한 간단한 피드백을 제공하는 ui입니다.
> Snackbar는 동작과 관련된 짧은 텍스트 한 줄과 단일 텍스트 액션(버튼)을 포함할 수 있습니다.

해당 포스팅에서는 **Scaffold**를 이용하지 않고 **Snackbar**를 보여주는 코드를 구현할 것이다.
**SnackbarHostState**를 이용해야지만 snackbar를 화면에 보일 수 있다는 점을 유의해야한다.

## **SnackbarHostState**

```kotlin
val snackbarHostState = remember {
    SnackbarHostState()
}
```

**snackbarHostState** 상태를 기억할 위의 코드를 작성한다. `snackbarHostState.showSnackbar()` 메서드를 통해 스낵바를 화면에 띄워야하기 때문이다.
<br>

### currentSnackbarData

**snackbarHostState** 를 클릭해보면

<div align="center">
  <img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/da3f26f5-caaa-406e-889c-dd4959afd57b">   
</div>

```kotlin
var currentSnackbarData by mutableStateOf<SnackbarData?>(null)
```

위와 같은 코드가 있는 것을 볼 수 있다. **currentSnackbarData** 은 스낵바가 현재 화면에 보여지는지에 대한 값으로 화면에 보여지지 않으면 **null**, 화면에 보여지면 스낵바의 객체가 저장된다.
<br>

여기서 알 수 있는 점은 **currentSnackbarData**은 상태 값으로 저장되어 있다는 점이고 스낵바가 보이고 안보이고에 따라 **currentSnackbarData**을 사용하는 컴포저블 객체들은 <span style="color:rgb(77,171,254)"> **리컴포지션** </span> 이 일어난다는 것이다.

## SnackbarHost

- **Custom한 snackbar** -> Snackbar()컴포저블 이용

  ```kotlin
  @Composable
  fun CustomSnackbarHost(snackbarHostState: SnackbarHostState, modifier: Modifier) {
      // Modifier 값을 Modifier.align(Alignment.BottomCenter) 이것으로 default 설정할 것
      SnackbarHost(
          hostState = snackbarHostState, modifier = modifier
      ) {
          Snackbar(
              backgroundColor = Color(0xFF55B938),
              contentColor = Color.White,
              modifier = Modifier.padding(12.dp),
              action = {
                  TextButton(
                      colors = ButtonDefaults.textButtonColors(contentColor = Color.White),
                      onClick = { it.performAction() },
                      content = { Text(text = "확인") }
                  )
              }
          ) {
              Row(verticalAlignment = Alignment.CenterVertically) {
                  Icon(imageVector = Icons.Default.TaskAlt, contentDescription = "")
                  Spacer(modifier = Modifier.width(8.dp))
                  Text(text = "Test")
              }
          }
      }
  }
  ```

  <div align="center">
  <img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/972b9343-fbc5-44e1-955d-c6bd9be44a78">   
  </div>

  당연히 모두가 알고있겠지만, `Modifier.align(Alignment.BottomCenter)`을 사용하기 위해서 `CustomSnackbarHost()` 컴포저블 상위의 `Box` 컴포저블이 있어야 한다.
  <br>

  앞에서 설정한 [**SnackbarHostState**](##SnackbarHostState) 를 `SnackbarHost()`의 매개변수로 넣어준다. 여기서 `SnackbarHost()` 컴포저블의 후행 람다슬롯의 `Snackbar()`을 지정하지 않으면 기본적인 배경색, 색상 등이 적용된다.
  <br>

  여기서 `it.performAction()`에 대해 많이 궁금할 수 있다. 여기서 **it**은 `SnackbarData` 객체이다. `SnackbarData.performAction()` 메서드를 통해 클릭 이벤트를 구현할 수 있다. 아래의 코드에서 살펴보자.

- **Default snackbar**

  ```kotlin
  @Composable
  fun CustomSnackbarHost(snackbarHostState: SnackbarHostState, modifier: Modifier) {
      // Modifier 값을 Modifier.align(Alignment.BottomCenter) 이것으로 default 설정할 것
      SnackbarHost(
          hostState = snackbarHostState, modifier = modifier
      )
  }
  ```

  <div align="center">
  <img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/a63ea7af-c0f5-4a45-8847-f8289b34903f">   
  </div>

  `SnackbarHost()` 컴포저블의 후행 람다슬롯을 비웠을 경우이다.

> 참고로 `@Composable CustomSnackbarHost()`만 만들었다고 snackbar가 보이지 않는다. 앞에서 얘기했듯이 `snackbarHostState.showSnackbar()` 코드가 필요하다.

## References

[Dialog, Toast, Snackbar](https://brunch.co.kr/@oemilk/91)  
[Compose Snackbar](https://velog.io/@dong-hyeok/Androidkotlin-JetPack-Compose-snackBar)
