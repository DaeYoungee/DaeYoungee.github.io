---
title: "[Kotlin] used Camera "
excerpt: "compose에서 사진을 찍어 uri를 가져오는 과정을 살펴보자."

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

이번 포스팅에서는 **Camera Permission**에 관해서는 다루지 않습니다. 카메라를 통해 사진을 저장하는 과정을 살펴볼 것이니 Camera Pemisson이 궁금하신 분은 [링크](https://daeyoungee.github.io/kotlin/CameraPermission/)를 통해 확인하세요

## AndroidManifest.xml

```kotlin
<!-- 카메라 권한 -->
<uses-feature
    android:name="android.hardware.camera"
    android:required="false" />

<uses-permission android:name="android.permission.CAMERA" />

<application>
  ...
  <provider
      android:name="androidx.core.content.FileProvider"
      android:authorities="com.example.planet.provider"
      android:grantUriPermissions="true"
      android:exported="false">
      <meta-data
          android:name="android.support.FILE_PROVIDER_PATHS"
          android:resource="@xml/filepaths" />
  </provider>
</application>
```

### filepaths.xml

**res/xml/filepaths.xml** 에 위치한다.

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-cache-path name="my_images" path="/"/>
</paths>
```

FileProvider를 사용하기 위해 `<provider>`를 설정한다. 여기서 주의해야할 점은 authorities 이다.
authorities은 **현재 프로젝트의 패키명 + .provider**로 설정해야한다.

## Context.createImageFile()

카메라는 사진을 찍고 uri를 리턴한다. uri에 지정한 이름과 uri가 만들어질 경로를 설정하는 코드이다.

```kotlin
fun Context.createImageFile(): File {
    // Create an image file name
    val timeStamp = SimpleDateFormat("yyyyMMdd_HHmmss").format(Date())
    val imageFileName = "JPEG_" + timeStamp + "_"
    val image = File.createTempFile(
        imageFileName, /* prefix */
        ".jpg", /* suffix */
        externalCacheDir      /* directory */
    )
    return image
}
```

## 실제 코드

```kotlin
@Composable
fun UseCamera() {
    DisposableEffect(Unit) {
        onDispose {
            val path = "/storage/emulated/0/Android/data/${BuildConfig.APPLICATION_ID}/cache/"
            val cashFile = File(path)
            val result = cashFile.allDelete()
        }
    }

    val context = LocalContext.current
    val file = context.createImageFile()
    val uri = FileProvider.getUriForFile(
        Objects.requireNonNull(context),
        BuildConfig.APPLICATION_ID + ".provider", file
    )
    var capturedImageUri by remember {
        mutableStateOf<Uri>(Uri.EMPTY)
    }


    val cameraLauncher =
        rememberLauncherForActivityResult(ActivityResultContracts.TakePicture()) { success ->
            // cameraLauncher.launch(uri)를 실행하면  내부 코드 실행
            if (success) capturedImageUri = uri
        }

    val cameraPermissionState = rememberPermissionState(Manifest.permission.CAMERA)

    CameraButton {
        if (cameraPermissionState.status.isGranted) {
            runCatching {
                  // 카메라 촬영 시작
                  cameraLauncher.launch(uri)
              }.onFailure { error ->
                Log.d(TAG, "error: ${error.message}")
                }

        } else {
            // 권한 부여
            cameraPermissionState.launchPermissionRequest()
        }
    }
}
```

**BuildConfig.APPLICATION_ID**은 반드시 현재 사용중인 프로젝트의 패키지명으로 와야한다.  
카메라를 통해 가져온 이미지는 "/storage/emulated/0/Android/data/${BuildConfig.APPLICATION_ID}/cache/" 경로에 저장된다. 사용자가 지우지 않으면 영구적으로 저장된다. 불필요한 리소스이므로 `DisposableEffect{}`를 통해 제거해줘야 한다.

### File.allDelete()

디렉토리 또는 파일을 삭제하고 싶으면 `file.delete()`를 사용하면 된다. 그러나 디렉토리 내부에 데이터가 있으면 삭제가 되지않는다.  
`File.allDelete()` 확장함수를 만들어 디렉토리 내부의 데이터를 제거해보자.

```kotlin
fun File.allDelete():Boolean? {
    var returnData = false
    val result = runCatching {     if (this.exists()) {
        val files = this.listFiles()
        if (files != null && files.size >0) {
            files.forEachIndexed { index, file ->
                returnData = file.delete()
            }
            return returnData
        } else {
            return false
        }
    } else {
        return false
    } }.onSuccess { return true }.onFailure { error ->
        Log.d(TAG, "error: ${error.message}")
        return false }

    return result.getOrNull()
}
```

### CameraButton

```kotlin
@Composable
fun CameraButton(onClick:() -> Unit) {
    Card(
        modifier = Modifier.size(50.dp),
        shape = CircleShape,
        colors = CardDefaults.cardColors(
            contentColor = Color.Black,
            containerColor = Color.White
        ),
        elevation = CardDefaults.elevatedCardElevation(4.dp)
    ) {
        Box(
            modifier = Modifier.fillMaxSize().clickable { onClick() }
        ) {
            Icon(
                imageVector = Icons.Default.CameraAlt,
                contentDescription = null,
                modifier = Modifier.align(Alignment.Center)
            )
        }
    }
}
```

## FileProvider

FileProvider가 뭔지 궁금해하시는 분이 계실것 같아서 설명하려고한다. 사실 내가 몰랐고 궁금해서 찾아봤다.
<br>

먼저 ContentProvider에 대해 알아야 할 필요가 있다. **ContentProvider**는 데이터를 캡슐화하여 다른 응용프로그램에 제공하는 Android 구성 요소이다. 여러 응용 프로그램간에 데이터를 공유해야하는 경우에만 필요하다.  
예를 들어 연락처의 데이터를 다른 응용 프로그램과 공유하게 되는 경우에도 사용되고 이 때 ContentProvider의 하위 클래스인 ContactsProvider를 사용한다.

> FileProvider는 ContentProvider의 하위 클래스이며 앱의 내부 파일을 공유하는 데 사용된다.

<span style="font-size : 12pt"> **FileProvider를 사용하는 방법** </span>

- AndroidManifest파일에서 FileProvider를 정의한다.
- FileProvider가 다른 응용 프로그램과 공유 할 모든 경로가 포함된 XML 파일을 만든다.

### FileProvider의 속성

AndroidManifest.xml에서 FileProvider에 대해 정의하려면 아래의 속성에 대해 숙지해야한다.

- android:name
- android:authorities
- android:grantUriPermissions
- android:exported

<span style="font-size : 11pt"> **android:name** </span>  
name에는 FileProvider가 있는 경로를 설정한다. 여기서는 "androidx.core.content.FileProvider"로 설정했다.

<span style="font-size : 11pt"> **android:authorities** </span>  
적어도 하나의 고유 권한을 반드시 정의해야한다. Android 시스템은 모든 제공자의 목록을 유지하며 권한별로 이를 구분한다. **권한은 애플리케이션 ID가 Android 애플리케이션을 정의하는 거서럼 FileProvider를 정의한다.**  
필자는 **현재 프로젝트의 패키명 + .provider**를 사용했다.  
ex) "com.example.planet.provider"

<span style="font-size : 11pt"> **android:grantUriPermissions** </span>  
FileProvider를 잠긴 방으로 생각하면 이 속성은 외부 앱에 임시 일회성 키를 제공한다. FileProvider에 접근할 permission이 없을 경우 일시적으로 데이터 접근 permission을 부여 받을 수 있다. 임시 접근 권한이라고 생각하면 될 것 같다.  
이 속성을 사용하면 앱의 내부 저장소를 안전하게 공유할 수 있다. android:grantUriPermissions의 값을 true로 사용하자.

<span style="font-size : 11pt"> **android:exported** </span>  
exported속성은 외부 어플리케이션의 접근 여부를 설정할 수 있다.  
이 속성을 이해하려면 FileProvider를 문이 잠긴 방으로 생각해보자. 값을 true로 설정하면 기본적으로 모든 사람에게 문이 열린다. 다른 모든 앱이 권한을 부여받지 않고 FileProvider를 사용할 수 있기 때문에 큰 보안문제가 발생할 수 있다.  
android:exported의 값을 false로 설정하자.

### filepaths.xml 생성

FileProvider를 사용할 때 이 하위 요소를 정의해야한다. FileProvider가 외부 앱과 공유할 수 있는 모든 데이터 경로가 포함 된 XML 파일의 경로를 정의해야한다.

XML 파일에는 루트로 `<paths>` 요소가 있어야한다. `<paths>` 요소에는 다음 중 하나일 수 있는 하나 이상의 하위 요소가 있어야한다.

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 내부 앱 스토리지, Context.getFilesDir() -->
    <files-path name="" path=""/>
    <!-- 내부 앱 캐시 스토리지,Context.getCacheDir () -->
    <cache-path name="" path=""/>
    <!-- 공용 외부 저장소,  Environment.getExternalStorageDirectory() -->
    <external-path name="" path=""/>
    <!-- 외부 앱 스토리지,  Context.getExternalFilesDir(null) -->
    <external-files-path="" path=""/>
    <!-- 외부 앱 캐시 스토리지,  Context.getExternalCacheDir() -->
    <external-cache-path name="my_images" path="/"/>
</paths>
```

## Reference

[medium.com](https://medium.com/@dheerubhadoria/capturing-images-from-camera-in-android-with-jetpack-compose-a-step-by-step-guide-64cd7f52e5de)  
[Android FileProvider](https://eunplay.tistory.com/81)
