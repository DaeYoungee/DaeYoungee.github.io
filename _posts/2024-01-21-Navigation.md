---
title: "[Kotlin] Navigation"
excerpt: "kotlin jetpack compose에서 화면이동을 알아보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Navigation]
sidebar:
  nav: "counts"

data: 2024-01-21
last_modified_at: 2024-01-21

published: true
---

## Navigation

Koltin Jetpack Compose에서 화면 이동은 `NavHost`컴포저블을 이용한다. `NavHost`컴포저블에 `navController`를 등록시키면 된다. 나는 보통 `BottomNavigation`에서 화면 단위로 이동할 때 사용한다.  
생각보다 간단하다. 코드로 알아보자.

## 종속성 추가(bundle.gradle.kts), module 단위

```kotlin
implementation("androidx.navigation:navigation-compose:2.7.6")
```

## NavHost

```kotlin
val navController = rememberNavController()

NavHost(navController = navController, startDestination = "first") {
	composable("first") {
		FirstScreen(navController = navController)
	}
	composable("second") {
		SecondScreen(navController = navController)
	}
	composable("third") {
		ThirdScreen(navController = navController, "ASd")
	}
}
```

`navController`를 `NavHost`에 등록시킨다. `startDestination` 매개변수는 처음 사용자에 보일 화면을 설정하는 것이다. 각 이동가능한 화면은 `composable {}` 로 묶어준다. `composable(”first”)`에 first는 라우터 경로이다.

## navController의 메서드

보통 버튼을 눌렀을 때 와 같이 onClick 의 이벤트 처리로 사용된다.

```kotlin
navController.navigate("first") // first 컴포저블로 이동
navController.navigateUp()      // 이전 컴포저블로 이동 (뒤로가기 기능)
navController.popBackStack()    // 이전 컴포저블로 이동 (뒤로가기 기능)
```

## Navigation에서 값 전달

composable에 인자에 쓰인 라우터 경로에 원하는 값을 {}안에 넣는다.  
웹사이트의 URL 같이 작성된 것을 볼 수 있다.  
버튼을 클릭했을때, value가 비어있을 때만, third에 value값을 가지고 컴포저블 전환이 된다.

```kotlin
NavHost(navController = navController, startDestination = "first") {
	...
	composable("third/{value}") { backStackEntry ->
		ThirdScreen(
			navController = navController,
			value = backStackEntry.argument?.getString(value) ?: "") // argument가 아니라 원하는 값을 전달해줘도 된다.
	}
}

...

@Composable
fun FirstScreen(navController:NavController) {
	val (value, setValue) = remember{ mutableStateOf("") }
	...
	Button(onClick={
		if(value.isNotEmpty()) {
			navController.navigate("third/${value}")
		}
	})
}
```

compoable 내부 {} 안에 들어오는 NavBackStackEntry 라는 객체가 있다.
이 객체를 이용하여 value값을 추출하다

```kotlin
backStackEntry.argument?.getString(value) ?: ""
```

?. 로 null이 아닌 안전한 호출을 하고 만약 null 이면 ?: 로 대신할 값을 적어준다.

## References

[안드로이드 공식 홈페이지 navigation](https://developer.android.com/jetpack/compose/navigation?hl=ko#kts)
