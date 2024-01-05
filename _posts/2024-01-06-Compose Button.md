---
title: "[Kotlin] Koltin Jetpack Compose Button 컴포저블"
excerpt: "Koltin Jetpack Compose에서 Button의 색상, 그라데이션, 그림자, 패딩 입히는 방법을 알아보자, "

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose, Button]
sidebar:
  nav: "counts"

data: 2024-01-06
last_modified_at: 2024-01-06

published: true
---

## Button 컴포저블의 content 패딩

`Modifier.padding()` 했을 때 패딩 값에는 변화가 없다. `PaddingValues()`을 이용해야 됨을 기억하자.

- **Modifier.padding()**
  ```kotlin
  Button(
      onClick = { /*TODO*/ },
      colors = ButtonDefaults.buttonColors(
          backgroundColor = Color.LightGray,
          contentColor = Color.White
      ),
      modifier = Modifier.padding(16.dp),
  ) {
      Text(text = "test")
  }
  ```
- **PaddingValues()**

  ```kotlin
  Button(
      onClick = { /*TODO*/ },
      colors = ButtonDefaults.buttonColors(
          backgroundColor = Color.LightGray,
          contentColor = Color.White
      ),
      contentPadding = PaddingValues(horizontal = 24.dp, vertical = 12.dp),

  ) {
      Text(text = "test")
  }
  ```

<div align="center">
  <img alt="Compose Button Padding 비교" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/923704e4-e97d-4e0e-a915-c22ec7ffafc5">   
</div>

첫 번쨰 버튼이 아무것도 적용하지 않은 원래의 버튼이며 두 번째 버튼이 `Modifier.padding()` 이용한 버튼이고 세 번째 버튼이 `PaddingValues()`을 이용한 것이다. 확실하게 패딩이 적용되는 것을 볼 수 있다.

## Button 컴포저블의 색상 입히기

Button 컴포저블에 배경색을 입힐 때, `Modifier.background()`을 사용하면 안된다. 아래의 사진을 보면 이해하기 쉬울 것이다. `ButtonDefaults.buttonColors()`을 이용해야 함을 기억하자.

- **ButtonDefaults.buttonColors()**

  ```koltin
  Button(
      onClick = { /*TODO*/ },
      colors = ButtonDefaults.buttonColors(
          backgroundColor = Color.LightGray,
          contentColor = Color.White
      ),
  ) {
      Text(text = "test")
  }
  ```

- **Modifier.background()**

  ```koltin
  Button(
      onClick = { /*TODO*/ },
      modifier = Modifier.background(color = Color.LightGray),
  ) {
      Text(text = "test")
  }
  ```

<div align="center">
  <img alt="Compose Button Padding 비교" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/41a487a4-09b6-4462-919c-9e7c32aec326">   
</div>

첫 번째 버튼은 `ButtonDefaults.buttonColors()` 이용해서 정상적으로 버튼에 backgroundColor와 contentColor를 입힌 모습이다. 두 번째 버튼은 `Modifier.padding()`을 이용했고 실제 버튼의 배경색에 가려져 잘 적용되지 않은 모습이다.

## Button 컴포저블에 그라데이션 적용하기

## References

[Compose Button 색상](https://kotlinworld.com/243?category=974080)
[Compose Material3 Version 확인, 공식 홈페이지](https://developer.android.com/jetpack/androidx/releases/compose-material3?hl=ko)  
[Compose Material3 사용방법, 공식 홈페이지](https://developer.android.com/jetpack/compose/designsystems/material3?hl=ko)
[ElevatedButton 컴포저블](https://semicolonspace.com/jetpack-compose-material3-elevatedbutton/)
