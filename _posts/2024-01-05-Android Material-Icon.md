---
title: "[Kotlin] Android Material Icon"
excerpt: "Koltin Jetpack Compose에서 Material Icon 사용방법을 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Icon]
sidebar:
  nav: "counts"

data: 2024-01-05
last_modified_at: 2024-01-05

published: true
---

## 라이브러리 설치

```koltin
implementation "androidx.compose.material:material-icons-extended:$compose_ui_version"
```

## 아이콘 사용사례

```koltin
Icon(
    imageVector = Icons.Default.VolumeUp,
    contentDescription = "Water Sound Check",
    tint = Color.White,
)
```

<div align="center">
  <img alt="구글 머티리얼 아이콘 종류" src="https://github.com/DaeYoungee/Sign-up_Firebase/assets/121485300/7cfd9078-4bb5-4643-b61e-11b7739c7607">   
</div>

## Reference

[구글 머테리얼 아이콘 종류](https://fonts.google.com/icons?selected=Material+Icons:volume_up:&icon.query=sound&icon.platform=android)
[머테리얼 아이콘 사용방법](https://velog.io/@gogumi4502/Android-Material-Icons-%EC%82%AC%EC%9A%A9%EB%B2%95)
