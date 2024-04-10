---
title: "[Kotlin] TextField의 커스텀 및 사용법. "
excerpt: "Jetpack Compose의 TextField의 대해 알아보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose, TextField]
sidebar:
  nav: "counts"

data: 2024-04-11
last_modified_at: 2024-04-11

published: true
---

Pigma에 설계된 UI대로 TextField의 custom하려고 구글링을 했다. 이 과정속에서 앞으로도 많이 쓰이고 알아두면 좋은 코드를 적어보려고 한다.

### 데모 영상

아래의 영상을 보고 알아야 할 사실이 5개가 있다.

1. custom을 위한 BasicTextField
2. 현재 글자 수 업데이트
3. focus된 TextField
4. focus가 되면서 올라가는 TextField
5. focus된 TextField 이 후, cursor가 text의 끝을 가리키도록 할 수 있게 TextFieldValue 사용

   <br>

![Screen Recording 2024-04-11 at 1 22 14 AM](https://github.com/DaeYoungee/Compose_study/assets/121485300/95e4e6ce-68e2-4dc1-beb1-817620d9227e)

## BasicTextField

TextField를 커스텀하기 위해서는 BasicTextField를 사용해야 한다.  
(원하는 디자인뿐만 아니라 TextField의 원하는 height를 설정하기 위해서는 BasicTextField를 사용해야 한다.) -> [참고 레퍼런스](https://stackoverflow.com/questions/67681416/jetpack-compose-decrease-height-of-textfield)

필자는 아래의 사진과 같이 카카오톡에서 닉네임을 변경될 때 보여지는 TextField를 만들고 싶었다.
![image](https://github.com/DaeYoungee/Compose_study/assets/121485300/648da2a5-eba8-4250-86a1-e65c01c6230b)

<br>

BasicTextField는 TextField, OutlineTextField 등 모든 TextField에 어머니인 존재이다.  
BasicTextField을 보면 필수적으로 구현해야하는 매개변수인 **decorationBox**을 볼 수 있다.  
**decorationBox**를 통해서 내가 원하는 디자인을 만들 수 있다. **decorationBox**에는 **innerTextField**라는 매개변수(컴포저블 함수 타입)를 필요로 하는데  
**innerTextField**는 실제로 내가 글자를 입력하고 화면에 보여지는 TextField라고 생각하면 된다.

```kotlin
BasicTextField(
    ...
    decorationBox = { innerTextField ->
        innerTextField()
        Column(modifier = Modifier.fillMaxWidth()) {
            Box(
                modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp),
            ) {
                Text(
                    text = trailingText,
                    modifier = Modifier.align(Alignment.CenterEnd),
                    style = LocalTextStyle.current.copy(
                        fontSize = 16.sp,
                        textAlign = TextAlign.End,
                        color = Color.White,
                        fontWeight = FontWeight.SemiBold
                    )
                )
            }
            Divider(
                thickness = 1.dp,
                color = Color.White,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 2.dp)
            )
        }
    }
)

```

## 현재 글자 수 업데이트

화면의 업데이트는 한번만 되는데 쓸데없이 두개의 상태값(현재 입력문자, 현재 입력된문자 수)이 변경될 필요는 없다고 생각했다. 따라서 `derivedStateOf`를 이용해서 현재 글자 수를 계산했다.

```kotlin
val maxNicknameTextLength = 20
var nicknameTextValue by mutableStateOf(TextFieldValue(text = "Happy Bean", selection = TextRange("Happy Bean".length)))

val nicknameTextLength by derivedStateOf {
    "${nicknameTextValue.text.length} / $maxNicknameTextLength"
}

/* ... */

BasicTextField(
    value = nicknameTextValue,
    onValueChange = {
        nicknameTextValue = it
    },
...
    decorationBox = { innerTextField ->
        innerTextField()
        Column(modifier = Modifier.fillMaxWidth()) {
            Box(
                modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp),
            ) {
                Text(
                    text = nicknameTextLength,
                    modifier = Modifier.align(Alignment.CenterEnd),
                    style = LocalTextStyle.current.copy(
                        fontSize = 16.sp,
                        textAlign = TextAlign.End,
                        color = Color.White,
                        fontWeight = FontWeight.SemiBold
                    )
                )
            }
            Divider(
                thickness = 1.dp,
                color = Color.White,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 2.dp)
            )
        }
    }
)
```

## TextField focus

처음 영상을 봤을 때 연필 icon을 클릭했을 때 TextField에 자동으로 focus 되면서 softKeyBoard가 올라왔다. 그에 관련된 코드를 지금부터 살펴보겠다.  
TextField에 focus를 주기 위해서는 `FocusRequester()` 객체가 필요하다.  
`modifier.focusRequester()`를 통해서 `FocusRequester()` 객체를 원하는 TextField에 등록시킨다.  
`focusRequester.requestFocus()`를 통해 등록된 TextField에 focus를 준다.

`LaunchedEffect(Unit)`를 통해 처음 **UserNicknameTextField**가 실행되면 0.3초 이후에 자동으로 focus 되게 했다.

```kotlin
@Composable
fun UserNicknameTextField(
    text: TextFieldValue,
    onValueChange: (TextFieldValue) -> Unit,
    modifier: Modifier = Modifier,
    trailingText: String,
    updateText: () -> Unit
) {
    val focusRequester = remember {
        FocusRequester()
    }
    LaunchedEffect(Unit) {
        delay(300)
        focusRequester.requestFocus()
    }

    BasicTextField(
        modifier = modifier.focusRequester(focusRequester),
        ...
    )
}

```

## TextField의 가시성 처리

아래의 사진은 [링크](https://dev.to/tkuenneth/keyboard-handling-in-jetpack-compose-2593)에서 가져온 사진이다.

![image](https://github.com/DaeYoungee/Compose_study/assets/121485300/dbcdd7a3-a883-4ac0-a3d2-6cb2d25df468)

TextField에 focus가 되면서 softKeyboard가 올라오고 softKeyboard가 TextField를 가리는 현상이다. 당연히 사용자는 불편함을 호소할 수 밖에 없다.

**보다 좋은 TextField의 가시성을 위해** Manifest.xml에 한줄만 추가하면 된다.

```xml
<activity
...
android:windowSoftInputMode="adjustResize" />
```

## TextFieldValue 사용

TextField에 임의의 text가 있다는 가정하에 자동으로 focus를 줬을 때 cursor가 맨 앞을 가리키는 현상이 나타났다. 딱히 문제는 없지만 사용자 입장에서 불편함을 느낀다고 생각하여 cursor가 text의 맨 뒤를 가리키게 끔 바꾸고자 하였다.  
[참고 레퍼런스](https://stackoverflow.com/questions/70908515/how-to-change-text-field-cursor-position-in-jetpack-compose)

![Screenshot 2024-04-11 at 2 33 34 AM](https://github.com/DaeYoungee/Compose_study/assets/121485300/ee0fcb3d-1ffb-49e9-bdf3-194b19557eea)

이 때 `TextFieldValue`를 사용하면 보다 쉽게 해결할 수 있다.

![image](https://github.com/DaeYoungee/Compose_study/assets/121485300/4cc3d81b-7afb-41d2-b3a4-0a0f87b3206f)

반드시 `selection = TextRange("Happy Bean".length)` 이 코드를 설정해줘야지만 cursor의 위치를 "Happy Bean"의 맨 뒤로 위치시켜준다.

```kotlin
var nicknameTextValue by mutableStateOf(TextFieldValue(text = "Happy Bean", selection = TextRange("Happy Bean".length))) // 임의의 text = "Happy Bean" 이 있다고 가정

BasicTextField(
    modifier = modifier.focusRequester(focusRequester),
    value = nicknameTextValue,
    onValueChange = {
        nicknameTextValue = it
    }
    /* ... */
)
```

## 기타(추가로 알아두면 좋음)

- TextField의 underLine 제거  
  TextField의 Indicator의 색상을 Color.Transparent로 바꿔주면 된다.

  ```kotlin
  TextField(
      ...
      colors = TextFieldDefaults.textFieldColors(
          focusedIndicatorColor = Color.Transparent,
          unfocusedIndicatorColor = Color.Transparent,
          textColor = Black
      ),
  )
  ```

## Reference

[TextField 가시성 처리](https://dev.to/tkuenneth/keyboard-handling-in-jetpack-compose-2593)  
[TextField cursor position 설정](https://stackoverflow.com/questions/70908515/how-to-change-text-field-cursor-position-in-jetpack-compose)  
[TextField의 underLine 제거](https://velog.io/@victorywoo/Compose-%EC%82%AC%EC%9A%A9%EB%B2%95-TextField-indicator)
[TextField의 height 변경](https://stackoverflow.com/questions/67681416/jetpack-compose-decrease-height-of-textfield)
