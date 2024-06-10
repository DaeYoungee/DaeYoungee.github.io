---
title: "[Kotlin] Compose Recompostion 최적화 "
excerpt: "Compose에서 recompostion 최적화 방법에 대해 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose, recompostion]
sidebar:
  nav: "counts"

data: 2024-06-11
last_modified_at: 2024-06-11

published: true
---

##

프로젝트가 커지면서 화면을 구성하고 변경될 때 버벅거리는 현상을 경험해 본적 있을 것이다. UX의 불편함을 해소하기 위해 recompostion 최적화에 대해 많이 찾아봤고 공부해본 내용을 기록하고자 한다.  
**compostion**은 처음 화면을 그리는(화면 구성) 작업, **recompostion**은 화면 구성이 이 후 재구성하는(화면을 다시 그림) 작업이다.  
compose에서는 앱의 성능을 향상시키기위해 불필요한 recompostion을 최소화해야 한다.

## recompostion

**recomposition**은 Composable 내부의 원소가 변경이 되었을 때 Compose가 다시 계산되어 UI가 화면에 다시 그려지는 것이다. Compose가 화면에 그려지기까지는 3단계를 거치게 된다.

> 1. **Composition**: Composable tree를 생성하고 어떠한 UI들을 화면에 표시할 것인지 결정하는 과정입니다. - <font style="color:#cccccc;">**what to show**</font>
> 2. **Layout** : UI를 배치할 위치를 결정합니다. 이 과정은 측정과 배치라는 두 단계로 구성되며 레아이웃 요소는 Composable tree에 있는 각 노드의 레이아웃 요소 및 모든 하위 요소를 2D 좌표로 측정하고 배치합니다. - <font style="color:#cccccc;">**Where to show**</font>
> 3. **Draw** : UI를 화면에 렌더링하는 과정입니다. - <font style="color:#cccccc;">**Draw to the screen**</font>

<br>

컴포저블의 내부 데이터가 변경되어 recompostion을 하는 상황일 때, **컴포지션 단계**에서 기존의 컴포저블과 비교했을 때 변화가 없다면 **컴포지션 단계**를 건너뛴다. 이는 성능 향상으로 이어진다.

## Reference

[recomposetion 1](https://wannabe-master.tistory.com/m/8)  
[recomposetion 2](https://medium.com/androiddevelopers/jetpack-compose-debugging-recomposition-bfcf4a6f8d37)  
[recomposetion 3](https://tourspace.tistory.com/536)
