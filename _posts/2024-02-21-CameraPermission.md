---
title: "[Kotlin] Camera Permission"
excerpt: "compose에서 Camera 권한 사용에 대해 알아보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose, Camera]
sidebar:
  nav: "counts"

data: 2024-02-21
last_modified_at: 2024-02-21

published: true
---

JetPack Compose에서 Permission을 요청하기 위해서는 [**Google의 accompanist**](https://github.com/google/accompanist/blob/main/sample/src/main/java/com/google/accompanist/sample/permissions/RequestPermissionSample.kt) 모듈을 설치해야한다.

### build.Gradle.kts

```kotlin
    implementation ("com.google.accompanist:accompanist-permissions:0.34.0")
```

## rememberPermissionState

permission 에 대한 dependency 가 추가되면 rememberPermissionState 함수를 사용할 수 있다.  
`cameraPermissionState.status.shouldShowRationale`은 유저가 권한 요청을 거절 했을 때 호출된다.

```kotlin
@OptIn(ExperimentalPermissionsApi::class)
@Composable
fun RequestPermission() {
    val cameraPermissionState = rememberPermissionState(android.Manifest.permission.CAMERA)
    val requestPermissionLauncher = rememberLauncherForActivityResult(ActivityResultContracts.RequestPermission()) { isGranted ->
        if (isGranted) {
            // 권한 부여
            cameraPermissionState.launchPermissionRequest()
            Log.d(TAG, "camera permission is ok")
        } else {
            // 권한 거부 처리, 처음 거절했을 때
            Log.d(TAG, "camera permission is reject")
        }
    }
    LaunchedEffect(cameraPermissionState) {
        if (cameraPermissionState.status.isGranted.not() && cameraPermissionState.status.shouldShowRationale) {
            // 필요한 경우 근거 표시
            // cameraPermissionState.status.shouldShowRationale 는 카메라 권한을 이미 거절했을 때 로그가 남음
            // 즉, 거절한 뒤 다시 앱을 실행했을 때 로그가 찍힘
            Log.d(TAG, "camera permission is reject(LaunchedEffect)")
        } else {
            requestPermissionLauncher.launch(android.Manifest.permission.CAMERA)
            Log.d(TAG, "camera permission is ok(LaunchedEffect)")
        }
    }
}
```

## 겪었던 이슈

`cameraPermissionState.launchPermissionRequest()` 는 권한 요청하는 메서드이다. 그냥 이 메서드를 사용할 경우 아래와 같은 오류가 뜬다.  
<br>
<span style="color:rgb(252,41,41); font-size : 18pt"> **ActivityResultLauncher cannot be null** </span>  
<br>
Activity가 아닌 Composable에서 바로 호출되기 때문에 이와 같은 오류가 발생한다고 생각한다.  
Button 컴포저블을 이용하면 아래의 코드처럼 오류없이 사용이 가능한다.

```kotlin
Button(onClick = { cameraPermissionState.launchPermissionRequest() }) {
    Text("권한 요청")
}
```

즉 `rememberLauncherForActivityResult`를 통해 **ManagedActivityResultLauncher** 객체를 만들고 `requestPermissionLauncher.launch(android.Manifest.permission.CAMERA)`으로 카메라 사용 권한을 요청한다.

## Reference

[Google의 accompanist 사용 예제](https://github.com/google/accompanist/blob/main/sample/src/main/java/com/google/accompanist/sample/permissions/RequestPermissionSample.kt)  
[Jetpack Compose에서 권한 요청](https://dongtrivia.com/entry/Android-JetPack-Compose-%EC%97%90%EC%84%9C-permission-%EC%9A%94%EC%B2%AD%ED%95%98%EA%B8%B0)  
[이슈 해결할 때 참고한 레퍼러스](https://medium.com/@rzmeneghelo/how-to-request-permissions-in-jetpack-compose-a-step-by-step-guide-7ce4b7782bd7)
