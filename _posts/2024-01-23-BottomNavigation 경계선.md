---
title: "[Kotlin] BottomNavigation 경계선"
excerpt: "BottomNavigation 경계선을 통해 UI 분리를 해보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, UI, Material3, BottomNavigation]
sidebar:
  nav: "counts"

data: 2024-01-23
last_modified_at: 2024-01-23

published: true
---

## BottomNavigation에서 border

BottomNavigation에 stroke를 설정해 BottomNavigation과 위의 UI를 구분하기 쉽게 설정할 수 있다. 그러나 일반적인 `Divider()` 컴포저블은 먹히지 않으므로 아래의 코드를 살펴보자.  
아래의 사진을 보면 border의 유뮤에 따라 UI가 달라 보일 수 있는 것을 알 수 있다.

### roundCorner 설정된 BottomNavigation

- border 없음
    <div align="center">
    <img alt="" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/6ac31d0b-1a63-4548-8cb0-d1806005603a">   
    </div>
    <br>

- border 있음
     <div align="center">
    <img alt="" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/4b1d377e-5ef9-4763-95ed-954879f00ab4">   
    </div>

### 일직선 BottomNavigation

- border 없음
    <div align="center">
    <img alt="" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/6ac31d0b-1a63-4548-8cb0-d1806005603a">   
    </div>
    <br>

- border 있음
     <div align="center">
    <img alt="" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/969ce6fe-6014-4672-9e58-ceb4c8439183">   
    </div>

### roundCorner 설정된 BottomNavigation 코드

- border 없음

```kotlin
NavigationBar(
    modifier = Modifier
        .fillMaxWidth()
        .drawBehind {
            drawRoundRect(
                color = color,
                cornerRadius = CornerRadius(20.dp.toPx(), 20.dp.toPx()),
                style = Stroke(width = 2f)
            )
        }
        .clip(shape = RoundedCornerShape(topStart = 20.dp, topEnd = 20.dp)),
    containerColor = Color.White,
    contentColor = Color.Black,
    tonalElevation = 100.dp
)
```

### 일직선 BottomNavigation 코드

```kotlin
NavigationBar(
    modifier = Modifier
        .fillMaxWidth()
        .drawBehind {
            drawLine(color = color, start = Offset(0f, 0f), end = Offset(size.width, 0f), strokeWidth = 1.dp.toPx())
        },
    containerColor = Color.White,
    contentColor = Color.Black,
)
```
