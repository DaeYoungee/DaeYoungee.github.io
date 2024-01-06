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

- **Modifier.padding()** --> 사용 ❌
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

  ```kotlin
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

- **Modifier.background()** --> 사용 ❌

  ```kotlin
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

그라이데이션을 적용하기 위해서는 `Modifier.background(brush = )`를 이용해야한다. 그러나 앞서 버튼의 색을 적용할 때 `Modifier` 를 사용하는 것은 바람직하지 않다고 설명했다.

<br>
어떻게 해야할까? 🧐

<br>   
정답은 버튼의 content 영역에 Row, Column, Box 등 레이아웃 컴포저블을 배치해서 modifier를 적용하는 것이다. 이 때 버튼의 색상은 투명으로 줘야하는 것을 잊지말자. 아래의 사진을 보면 쉽게 이해갈 것이다.

- **Button 컴포저블의 매개변수**  
  Button 컴포저블의 매개변수중 content는 컴포저블 함수의 타입인 것을 알 수 있다.
  <div align="center">
    <img alt="Compose Button Padding 비교" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/b6a11010-7c92-4b30-836d-0d42dd024bca">   
  </div>
- **content 영역**  
  this: RowScope 라고 써진부분이 cotent 영역이다.
  <div align="center">
    <img alt="Compose Button Padding 비교" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/96488ccc-9218-41a2-8fa2-6c627637b2a8">   
  </div>
- 실제 코드

  ```kotlin
  Button(
      onClick = { /*TODO*/ },
      modifier = Modifier
          .fillMaxWidth()
          .height(90.dp),
      colors = ButtonDefaults.buttonColors(backgroundColor = Color.Transparent),  //  버튼 배경색 투명화
      contentPadding = PaddingValues(),
      shape = RoundedCornerShape(20.dp)  // 모서리 둥글게 만들기
  ) {
      Row(
          modifier = Modifier
              .fillMaxWidth()    // 필수!
              .fillMaxHeight()   // 필수!
              .clip(shape = RoundedCornerShape(20.dp)) // 버튼이랑 모서리 각도 똑같이 적용할 것!
              .background(
                  brush = Brush.verticalGradient(
                      colors = listOf(
                          colorResource(id = R.color.main_blue),
                          colorResource(id = R.color.opaque_blue)
                      )
                  )
              ),  // 실제 버튼 그라데이션 적용하는 부분
          horizontalArrangement = Arrangement.Center,
          verticalAlignment = Alignment.CenterVertically
      ) {
        // 사용자들이 알아서 커스텀해서 적용할 것
          Icon(
              imageVector = Icons.Default.VolumeUp,
              contentDescription = "Water Sound Check",
              tint = Color.White,
              modifier = Modifier
                  .padding(end = 16.dp)
                  .size(30.dp)
          )
          Text(
              text = "Water Sound Check",
              color = Color.White,
              fontWeight = FontWeight.Bold,
              fontSize = 20.sp
          )
      }
  }
  ```

  `Color.Transparent` 는 투명한 색상을 의미하고 `Brush.verticalGradient()` 는 세로로 그라데이션 색상을 적용함을 의미한다.  
   당연하게도 버튼과 **shape**가 같아야 하며, `fillMaxWidth()`, `fillMaxHeight()` 메서드를 통해 버튼과 크기를 동일하게 맞춘다.

  > 📌 정리
  >
  > 1.  버튼과 Shape을 동일하게 한다.
  > 2.  버튼과 Size를 동일하게 한다.
  > 3.  버튼의 배경은 투명으로 한다.

  <br>

- 결과
  <div align="center">
    <img alt="Compose Button Padding 비교" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/98602adc-0dac-4e20-add7-225fda9e1679">   
  </div>

## References

[Compose Button 색상](https://kotlinworld.com/243?category=974080)  
[Compose Material3 Version 확인, 공식 홈페이지](https://developer.android.com/jetpack/androidx/releases/compose-material3?hl=ko)  
[Compose Material3 사용방법, 공식 홈페이지](https://developer.android.com/jetpack/compose/designsystems/material3?hl=ko)  
[ElevatedButton 컴포저블](https://semicolonspace.com/jetpack-compose-material3-elevatedbutton/)
