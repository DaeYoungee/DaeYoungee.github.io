---
title: "[Kotlin] Material3, BottomNavigation"
excerpt: "Material3에서 BottomNavigation을 사용법을 알아보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Material3, BottomNavigation]
sidebar:
  nav: "counts"

data: 2024-01-21
last_modified_at: 2024-01-21

published: true
---

## BottomNavigation

kotlin jetpack compose에서 Material3에서 BottomNavigation를 구현

### BottomNavigation으로 보는 화면 이동

![navigation](https://github.com/DaeYoungee/Compose_study/assets/121485300/8cc5f86c-0104-428b-b6f8-aab1893373cb)

## sealed Class

navigation으로 화면이동을 할 때 route를 설정하게 되는데 이는 보통 `sealed Class`로 설정하게 된다.

```kotlin
const val HOME = "homeScreen"
const val STATISTIC = "statisticScreen"
const val SETTING = "settingScreen"

sealed class BottomNavItem(
    val screenRoute: String,
    var bottomIcon: ImageVector,
    val bottomTitle: String,
    var color: Color,
) {
    object HomeScreen : BottomNavItem(
        screenRoute = HOME,
        bottomIcon = Icons.Outlined.Home,
        bottomTitle = "Home",
        color = Color(0xFF007AFF),
    )
    object StatisticScreen : BottomNavItem(
        screenRoute = STATISTIC,
        bottomIcon = Icons.Outlined.Equalizer,
        bottomTitle = "Statistics",
        color = Color.Black,
    )
    object SettingScreen : BottomNavItem(
        screenRoute = SETTING,
        bottomIcon = Icons.Outlined.Person,
        bottomTitle = "Setting",
        color = Color.Black,
    )
}
```
