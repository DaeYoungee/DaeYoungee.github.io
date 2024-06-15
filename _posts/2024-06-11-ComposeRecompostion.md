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

프로젝트가 커지면서 화면을 구성하고 변경될 때 버벅거리는 현상을 경험해 본적 있을 것이다. UX의 불편함을 해소하기 위해 recompostion 최적화에 대해 많이 찾아봤고 공부해본 내용을 기록하고자 한다.  
**compostion**은 처음 화면을 그리는(화면 구성) 작업, **recompostion**은 화면 구성이 이 후 재구성하는(화면을 다시 그림) 작업이다.  
compose에서는 앱의 성능을 향상시키기위해 불필요한 recompostion을 최소화해야 한다.

## recompostion

**recomposition**은 Composable 내부의 원소가 변경이 되었을 때 Compose가 다시 계산되어 UI가 화면에 다시 그려지는 것이다. Compose가 화면에 그려지기까지는 3단계를 거치게 된다.

> 1. **Composition**: Composable tree를 생성하고 어떠한 UI들을 화면에 표시할 것인지 결정하는 과정입니다. - <font style="color:#cccccc;">**what to show**</font>
> 2. **Layout** : UI를 배치할 위치를 결정합니다. 이 과정은 측정과 배치라는 두 단계로 구성되며 레아이웃 요소는 Composable tree에 있는 각 노드의 레이아웃 요소 및 모든 하위 요소를 2D 좌표로 측정하고 배치합니다. - <font style="color:#cccccc;">**Where to show**</font>
> 3. **Draw** : UI를 화면에 렌더링하는 과정입니다. - <font style="color:#cccccc;">**Draw to the screen**</font>

<br>

컴포저블의 내부 데이터가 변경되어 recomposition을 하는 상황일 때, **컴포지션 단계**에서 기존의 컴포저블과 비교했을 때 변화가 없다면 **컴포지션 단계**를 건너뛴다. 이는 성능 향상으로 이어진다.

### 불필요한 recomposition

compose에서는 불필요한 recomposition울 줄이는 것이 compose의 성능 개선에 많은 도움을 준다.  
불필요한 recomposition이란 값은 그대로이고 UI가 변하지 않는 것이라 한다.

## Compose의 Stability

Compose는 **stable**, **unstable** 타입 2가지로 구분된다.  
stable: Compose가 immutable하거나 recomposition이 일어났을 때 내부의 어떤 값이 변했는지 파악할 수 있는 경우,  
unstable: recomposition이 일어났을 때 내부의 어떤 값이 변했는지 파악할 수 없는 경우

타입이 stable이냐, unstable이냐에 따라 recomposition을 일으키는 결정적인 요인이 된다.  
만약 Compose 컴포넌트가 stable하고 내부의 값이 변경되지 않는다면 해당 컴포넌트는 recomposition이 일어나지 않는다. 그러나 Compose 컴포넌트가 unstable하고 내부의 값이 변경되지 않았어도 부모 컴포넌트가 recomposition 될 때 자식 컴포넌트도 recomposition된다.

> **즉, 값이 변하지 않아도 unstable하면 recomposition이 발생하기 때문에 이는 성능 저하로 이루어질 수 있다.**

## Compose에서 구현

Compose 컴파일러는 코드를 읽으면서 각 컴포넌트에 태그를 표시한다. 해당 태그를 보고 recomposition을 수행할지 여부를 정한다.

<br>

다음은 Composable 함수에 표시하는 태그이다. 둘 중 하나만 표시할 수도 있고 아무것도 표시하지 않는 경우도 있다.

- Skippable: 해당 태그가 있는 컴포넌트는 Compose가 recomposition을 일으키지 않고 스킵한다. 해당 컴포넌트의 모든 인자 값들이 이전의 값과 새롭게 바뀐 값이 동일한 경우에 해당 태그가 달린다.
- Restartable: 해당 컴포넌트를 기준으로 스코프를 만들어 recomposition이 일어난다. 즉 state가 변할 수 있고 recomposition의 시작점이다.

Compose의 Parameter 구분

- Immutable  
  Compose에서는 모든 프로퍼티들이 immutable이고 메서드들이 참조 투명하면 immutable 태그를 붙입니다. 모든 primitive 타입(String, Int, Float 등)은 immutable로 표시된다. val 형태인 data class도 있다.

- Stable  
  생성 후 프로퍼티가 변경될 수 있는 유형으로, 즉 값이 바뀔 수는 있지만 런타임 중 프로퍼티가 변경된다면 Compose가 변경 사항을 추적할 수 있는 컴포넌트, 대표적으로 `State<T>`가 있다.
- Unstable
  Immutable과 Stable에 속하지 않는 유형으로 recomposition을 유발한다.

## 불필요한 Recomposition 줄이기

1.  immutable collection 사용
    Compose는 List, Set, Map과 같은 collection을 값이 바뀔 수 있다고 생각하여 unstable한 객체라고 판판한다. 즉, 우리가 흔히 사용하는 data class 안에 List 타입의 프로퍼티가 있다면 val로 선언되어 해당 data class는 unstable한 객체가 된다.

    nstable한 Collection 객체들을 stable하게 사용하고 싶다고 compose에게 알려주기 위해서는 @immutable이나 @Stable 어노테이션을 붙이면 된다. 단 개발자가 임의로 @Immutable, @Stable 붙여도 컴파일러가 unstable하다고 판단한다면 recomposition이 일어날 수 있다.  
    이 뿐 아니라 List, Set, Map과 같은 Collection을 ImmutableList, ImmutableSet, ImmutableMap 과 같은 immutable collection로 사용해서 compose에게 stable하다고 알려줄 수 있다.

    immutable collection을 사용하기 위해서는 dependency를 추가해야 한다. -> 변경되는 버전이나 최신 사항([깃허브 힝크](https://github.com/Kotlin/kotlinx.collections.immutable)) 참조하면 된다.

    ```kotlin
    implementation ("org.jetbrains.kotlinx:kotlinx-collections-immutable:0.3.6")
    ```

    사용방법은 아래와 같다.

    ```kotlin
    val examples: ImmutableList<Example> = persistentListOf()
    ```

    `persistentListOf()`는 empty한 ImmutableList를 반환한다.

2.  Painter 클래스 사용 X  
    이미지를 그리는 Painter 클래스는 unstable하다. 이미지의 painter 인자에 Painter 클래스를 활용하면 이미지가 변하지 않는 상황에서도 불필요한 recompostion이 일어날 수 있다. Painter 클래스 대신 painterResource를 이용해 stable한 Int 값인 resourceId를 넘겨주면 불필요한 recomposition을 막을 수 있다.

    ```kotlin
    Image(
     painter = painterResource(id = R.drawable.example_img),
     contentDescription = "Example Image"
    )
    ```

    만약, svg 파일을 사용하는 경우 ImageVector 클래스를 이용할 수 있다. ImageVector class는 내부적으로 @Immutable 어노테이션을 가지고 있기 때문에 stable하다.

3.  반복문(for, forEach), Lazy Composable에 key값 사용  
     아래와 같은 for문이 있을 때 원소가 추가되거나 삭제될 경우 recompostion이 어떻게 발생할까?

    ```kotlin
    @Composable
    fun MoviesScreen(movies: List<Movie>) {
       Column {
           for (movie in movies) {
               MovieOverview(movie)
           }
       }
    }
    ```

    정답은 원소가 추가되는 위치에 따라 다르다. 원소가 마지막에 추가되면 기존에 있는 원소들의 위치가 그대로이기 때문에 MovieOverview 인스턴스를 재활용할 수 있어 recomposition이 일어나지 않는다.

    반면 원소가 맨 앞에 추가되거나 정렬이 바뀌거나 중간에 원소가 추가되어 위치가 바뀌게 되는 원소들은 내부의 movie 값이 동일해도 MovieOverview가 recomposition이 일어나게 된다. **"따라서 내부의 값이 동일하면 Skippable하게 만들어야하고 이는 key를 이용해야 한다."**

    ```kotlin
    @Composable
     fun MoviesScreen(movies: List<Movie>) {
       Column {
           for (movie in movies) {
               key(movie.id) {
                   MovieOverview(movie)
               }
           }
       }
    }
    ```

    위의 코드와 같이 moive를 식별할 수 있는 id를 key로 주어기게 된다면 각각의 MovieOverview가 movie의 id와 매핑되어 composition 트리내에 인스턴스를 재활용하고 재정렬해서 불필요한 recompositon을 줄일 수 있다.

4.  외부 모듈에 위치한 class는 unstable로 처리된다.  
     Compose compiler 동작하지 안는 외부 모듈에 위치한 class들은 전부 unstable로 처리된다. 특히 `LocalDateTime`을 사용하는 경우 그렇다.

    ```kotlin
    data class FoodInfo(val name: String, val timestamp: LocalDateTime)

    // Compose compiler report
    unstable class FoodInfo {
        stable val name: String
        unstable val timestamp: LocalDateTime
        <runtime stability> = Unstable
    }
    ```

    LocalDateTime을 사용하면서 FoodInfo class 자체가 unstable로 변경되었다. 이를 해결하기 위해 @Immutable 또는 @Stable annotation을 이용하여 강제로 stability를 줄수 있지만. 이러한 방법은 지양되는 방법이다. 데아터가 변경되어 recomposition이 일어날 시 skip될 수 있기 때문이다.

    ```kotlin
    // 변경전
    data class FoodInfo(val name: String, val timestamp: LocalDateTime)

    // 변경후
    data class FoodUiInfo(val name: String, val timestamp: Long)

    // mapper의 구성
    fun FoodInfo.toImmutable(): FoodUiInfo =
    FoodUiInfo(this.name, Timestamp.valueOf(this).getTime())

    ```

\* 참조 투명한 함수: 함수나 메서드가 주어진 입력에 대해 항상 동일한 출력을 생성, 부작용(side effects)가 없는 성질을 의미

- 참조 투명한 함수

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```

- 참조 투명하지 않은 함수

```kotlin
var total = 0
fun addWithSideEffect(a: Int): Int {
    total += a
    return total
}
```

## Reference

[그림으로 이해하는 컴포지션 3단계](https://developer.android.com/develop/ui/compose/phases?hl=ko)  
[recomposetion 1](https://wannabe-master.tistory.com/m/8)  
[recomposetion 2](https://medium.com/androiddevelopers/jetpack-compose-debugging-recomposition-bfcf4a6f8d37)  
[recomposetion 3](https://tourspace.tistory.com/536)
