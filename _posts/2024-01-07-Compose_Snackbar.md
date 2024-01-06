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
<br>

```kotlin
var currentSnackbarData by mutableStateOf<SnackbarData?>(null)
```

위와 같은 코드가 있는 것을 볼 수 있다. **currentSnackbarData** 은 스낵바가 현재 화면에 보여지는지에 대한 값으로 화면에 보여지지 않으면 **null**, 화면에 보여지면 스낵바의 객체가 저장된다.  
`snackbarHostState.currentSnackbarData` 코드를 Log로 찍어보면 알 수 있다.
<br>

여기서 알 수 있는 점은 **currentSnackbarData**은 상태 값으로 저장되어 있다는 점이고 스낵바가 보이고 안보이고에 따라 **currentSnackbarData**을 사용하는 컴포저블 객체들은 <span style="color:rgb(77,171,254)"> **리컴포지션** </span> 이 일어난다는 것이다.

## SnackbarHost

**SnackbarHost**를 통해 실제 화면에 보여질 snackbar를 나타내는 컴포저블 함수이다.  
먼저 **SnackbarHost**의 구조에 대해 알아보자.

<div align="center">
<img alt="SnackbarHost" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/985b2194-6f4f-41bb-81c2-542e4fd926ab">   
</div>

**snackbar** 매개변수의 타입은 컴포저블 함수 타입이고 컴포저블 함수의 매개변수로는 **SnackbarData**가 필요하다.  
<span style="color:rgb(77,171,254)"> **SnackbarData** </span>을 쉽게 설명하면 스낵바의 우측에 표시되는 컴포넌트이다. 아래의 사진속에서는 "완료"부분이 된다.

<div>

## Snackbar

조금더 자세히 알아보기 위해 **Snackbar**의 구조를 알아보자.

<div align="center">
<img alt="Snackbar" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/4a8dbf75-fb79-4742-ab64-916cf29228eb">   
</div>

`Snackbar()` 컴포저블의 매개변수에서 **content**는 좌측에 나타나는 컴포넌트, **action**은 우측에 나타나는 컴포너트이다. 아래의 그림으로 설명하면 **content**는 "snackBar show!!" 이고 **action**은 "확인"이다.  
**action**의 데이터와 이벤트 처리는 **SnackbarData**을 통해 구현되는 것을 볼 수 있다. 그래서 아래의 <span style="color:rgb(77,171,254)"> **Custom한 snackbar** </span>에서 `onClick = {it.performAction()}` 을 이용해 **action**의 이벤트를 처리한 것이다.<br><br>

<div align="center">
<img alt="Snackbar" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/8ce56dcc-5c34-408b-a374-c9982dc1cae8">   
</div>
<br>

## snackbar의 custom 유무

snackbar의 custom 유무를 사진으로 한눈에 알아보자.

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

  앞에서 설정한 [**SnackbarHostState**](#snackbarhoststate) 를 `SnackbarHost()`의 매개변수로 넣어준다. 여기서 `SnackbarHost()` 컴포저블의 후행 람다슬롯의 `Snackbar()`을 지정하지 않으면 기본적인 배경색, 색상 등이 적용된다.
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

> 참고로 `@Composable CustomSnackbarHost()`만 만들었다고 **snackbar**가 보이지 않는다. 앞에서 얘기했듯이 `snackbarHostState.showSnackbar()` 코드가 필요하다.

## snackbarHostState.showSnackbar()

`showSnackbar()` 메서드는 suspend 함수이다. 즉, snackbar를 화면에 띄우기 위해서는 Coroutine 비동기 처리를 해야한다. Composable 함수에서는 `rememberCoroutineScope()`을 이용한다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/f8d9d4d7-46c9-4271-954d-b4e1a2512e40">   
</div>

## 전체 코드 및 설명

```kotlin
@Preview(showBackground = true)
@Composable
fun Test1() {

    // var currentSnackbarData by mutableStateOf<SnackbarData?>(null)
    // 스낵바호스트를 클릭해 보면 위 와 같은 코드가 있다.
    // currentSnackbarData는 현재 스낵바가 떠 있으면 값이 들어오고 없으면 null이다.
    val snackbarHostState = remember {
        SnackbarHostState()
    }

    val corutineScope = rememberCoroutineScope()

    // 스낵바의 상태에 따라 버튼의 text를 변경하는 메서드이다.
    // SnackbarData에 접근하여 스낵바가 올라와 있는지 아닌지에 따라 text를 변경한다.
    val buttonTitle: (SnackbarData?) -> String = { snackbarData ->
        if (snackbarData != null) {
            "스낵바 숨기기"
        } else {
            "스낵바 보여주기"
        }
    }

    // 스낵바의 상태에 따라 버튼의 color를 변경하는 메서드이다.
    // SnackbarData에 접근하여 스낵바가 올라와 있는지 아닌지에 따라  color를 변경한다.
    val buttonColor: (SnackbarData?) -> Color = { snackbarData ->
        if (snackbarData != null) {
            Color.Blue
        } else {
            Color.Red
        }
    }

    Box(
        modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.Center
    ) {
        // 버튼 클릭시 코루틴 스코프 안에서 실행
        Button(
            colors = ButtonDefaults.buttonColors(
                backgroundColor = buttonColor(snackbarHostState.currentSnackbarData),
                contentColor = Color.White
            ), onClick = {
                Log.d("snackBar", "snackBar: 스낵바 버튼 클릭")
                // 스낵바가 이미 올라와 있는데 스낵바를 띄우는 버튼을 또 클릭하면 스낵바를 내린다.
                if (snackbarHostState.currentSnackbarData != null) {
                    Log.d("snackBar", "snackBar: 이미 스낵바가 있음")
                    snackbarHostState.currentSnackbarData?.dismiss()
                    return@Button
                }

                corutineScope.launch {
                    // snackBar의 결과를 받을 수 있음
                    // 이 결과는 스낵바가 닫아졌는지? 스낵바의 버튼이 눌러졌는지?
                    snackbarHostState.showSnackbar(
                        "snackBar show!!", "확인", SnackbarDuration.Short // 스낵바 보여주는 시간
                    ).let {
                        when (it) {
                            SnackbarResult.Dismissed -> {
                                Log.d("daeYoung", "snackBar: 스낵바 닫아짐")
                                Log.d("daeYoung", "snackbarData: ${snackbarHostState.currentSnackbarData}")
                            }
                            // 스낵바에 있는 버튼이 눌러졌을 때 로직처리 하는 부분
                            SnackbarResult.ActionPerformed -> {
                                Log.d("daeYoung", "snackBar: 확인 버튼 눌러짐")
                                Log.d("daeYoung", "snackbarData: ${snackbarHostState.currentSnackbarData}")
                            }
                        }
                    }
                }
            }) {
            Text(buttonTitle(snackbarHostState.currentSnackbarData))
        }

        // 스낵바가 보여지는 부분은 따로 지정해 주어야 함
        CustomSnackbarHost(snackbarHostState = snackbarHostState, modifier = Modifier.align(Alignment.BottomCenter))
    }
}
```

이 중에서 `snackbarHostState.showSnackbar().let{}` 부분만 주의 깊게 보자.  
**SnackbarResult**를 반환한다. 즉, 앞서 설명한 action 버튼을 눌렀을 때 이벤트를 처리하거나 snackbar가 시간이 지나 자동으로 꺼졌을 때 이벤트를 처리한다.

- **SnackbarResult.Dismissed**  
  snackbar가 시간이 지나 스스로 사라졌을 때 이벤트 처리
- **SnackbarResult.ActionPerformed**  
  snackbar의 action 부븐을 클릭했을 때 이벤트 처리

### 커스텀으로 Snackbar를 만들 때 주의사항

`snackbarHostState.showSnackbar()`을 사용해서 스낵바를 화면에 띄울 때 [**SnackbarHost**](#snackbarhost)을 이용해 [**Snackbar**](#snackbar)를 커스텀했다면 `snackbarHostState.showSnackbar()`내의 매개변수의 message와 actionLabel은 의미가 없다.  
결국, 커스텀한 content와 action으로 덮어지기 때문이다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/272d5252-50d9-4a2f-b622-7eb8b0884e4c">   
</div>

## References

[Dialog, Toast, Snackbar](https://brunch.co.kr/@oemilk/91)  
[Compose Snackbar](https://velog.io/@dong-hyeok/Androidkotlin-JetPack-Compose-snackBar)
