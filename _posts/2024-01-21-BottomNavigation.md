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
last_modified_at: 2024-01-23

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
) {
    data object HomeScreen : BottomNavItem(
        screenRoute = HOME,
        bottomIcon = Icons.Outlined.Home,
        bottomTitle = "Home",
    )
    data object StatisticScreen : BottomNavItem(
        screenRoute = STATISTIC,
        bottomIcon = Icons.Outlined.Equalizer,
        bottomTitle = "Statistics",
    )
    data object SettingScreen : BottomNavItem(
        screenRoute = SETTING,
        bottomIcon = Icons.Outlined.Person,
        bottomTitle = "Setting",
    )
}
```

## NavHost

`NavHost`와 `navController`를 이용해서 실제 이동할 경로를 설정하는 컴포저블 함수이다.

```kotlin
@Composable
fun NavigationGraph(
    navController: NavHostController,
    recordViewModel: RecordViewModel,
    vicoViewModel: VicoViewModel
) {
    NavHost(navController = navController, startDestination = BottomNavItem.HomeScreen.screenRoute) {
        composable(BottomNavItem.HomeScreen.screenRoute) {
            if (!recordViewModel.isResult.value) {
                HomeScreen(viewModel = recordViewModel)
            } else {
                ResultScreen(viewModel = recordViewModel)
            }
        }
        composable(BottomNavItem.StatisticScreen.screenRoute) {
            StatisticesScreen(viewModel = vicoViewModel)
        }
        composable(BottomNavItem.SettingScreen.screenRoute) {

        }
    }
}
```

## BottomNavigation

실제 사용자에게 보여질 BottomNavigation의 아이콘, 색상, 클릭시 이벤트를 설정하는 컴포저블 함수이다.
나중에 Scaffold에 등록하게 된다.

```kotlin
@Composable
fun BottomNavigation(navController: NavHostController, viewModel: RecordViewModel) {
    val items = listOf<BottomNavItem>(
        BottomNavItem.StatisticScreen,
        BottomNavItem.HomeScreen,
        BottomNavItem.SettingScreen,
    )
    val color = colorResource(id = R.color.border_grey)
    NavigationBar(
        modifier = Modifier
            .fillMaxWidth()
            .drawBehind {
                drawRoundRect(
                    color = color,
                    cornerRadius = CornerRadius(20.dp.toPx(), 20.dp.toPx()),
                    style = Stroke(width = 1f)
                )
            }
            .clip(shape = RoundedCornerShape(topStart = 20.dp, topEnd = 20.dp)),
        containerColor = Color.White,
        contentColor = Color.Black,
        tonalElevation = 100.dp
    ) {
        val navBackStackEntry by navController.currentBackStackEntryAsState()
        val currentRoute = navBackStackEntry?.destination?.route
        items.forEach { item ->
            val selected = item.screenRoute == currentRoute
            NavigationBarItem(
                icon = {
                    Icon(
                        imageVector = item.bottomIcon,
                        contentDescription = item.bottomTitle,
                        modifier = Modifier.size(26.dp),
                    )
                },
                label = {
                    Text(
                        text = item.bottomTitle,
                        fontSize = 9.sp,
                    )
                },
                selected = selected,
                colors = NavigationBarItemDefaults.colors(
                    selectedIconColor = Color(0xFF007AFF),
                    selectedTextColor = Color(0xFF007AFF),
                    unselectedIconColor = Color.Black,
                    unselectedTextColor = Color.Black,
                    indicatorColor = Color.White
                ),
                alwaysShowLabel = true,
                onClick = {
                    navController.navigate(item.screenRoute) {
                        navController.graph.startDestinationRoute?.let {
                            popUpTo(it) { saveState = true }
                        }
                        launchSingleTop = true
                        restoreState = true
                    }
                },
            )
        }
    }
}

```

파란색은 `NavigationBar`, 빨간색은 `NavigationBarItem`로 알기 쉽게 사진으로 표현했다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/71ad8baa-d6b1-4d34-b87e-3791ad5c5d1c">   
</div>

### NavigationBar

`Modifier.drawBehind {}` 은 BottomNavigation에서 위의 선을 그어 상단 UI와 구분 짓기 위해 사용했다.

아래는 현재 사용자에게 보여지는 화면의 라우터를 알려주는 코드이다.

```kotlin
val navBackStackEntry by navController.currentBackStackEntryAsState()
val currentRoute = navBackStackEntry?.destination?.route
```

### NavigationBarItem

Color 매개변수를 보면 indicatorColor를 설정한 것을 볼 수 있다. indicatorColor를 Color.White로 줘서 터치했을 때 반투명하게 나타나는 indicator 효과를 제거한 것이다.

```kotlin
navController.navigate(item.screenRoute) {
    navController.graph.startDestinationRoute?.let {
        popUpTo(it) { saveState = true }
    }
    launchSingleTop = true
    restoreState = true
}
```

`onClick()` 이벤트로 터치시 `navController.navigate()`를 통해 해당 라우터에 설정된 screen으로 이동한다. `launchSingleTop = true`를 통해 인스턴스 하나만 만들어지게 하였고, `restoreState=true`를 통해 버튼을 재클릭했을 때 이전 상태가 남아있게 했다.

## MainActivity

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    private val recordViewModel by viewModels<RecordViewModel>()
    private val vicoViewModel by viewModels<VicoViewModel>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val navController = rememberNavController()

            WaterSaviorTheme {
                Scaffold(bottomBar = { BottomNavigation(navController = navController, viewModel = recordViewModel) }) {
                    Box(modifier = Modifier.padding(it)) {
                        NavigationGraph(recordViewModel = recordViewModel, vicoViewModel = vicoViewModel, navController = navController)
                    }
                }
            }
        }
    }
}
```

## References

[Material에서 BottomNavigation](https://velog.io/@chuu1019/Android-Jetpack-Compose-Bottom-Navigation-%EB%A7%8C%EB%93%A4%EA%B8%B0)  
[Material3에서 BottomNavigation Indicator 효과 제거](https://rkdrkd-history.tistory.com/70)
