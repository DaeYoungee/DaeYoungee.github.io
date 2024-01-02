---
title: "[Kotlin] DataStore"
excerpt: "코틀린 제트팩 컴포즈에서 DataStore가 어떻게 사용되는지 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin]
sidebar:
  nav: "counts"

data: 2023-11-27
last_modified_at: 2023-11-27

published: true
---

## SharedPreferences

- Key, Value 형태로 이용
- String, Int, Float, Boolean과 같은 원시형 데이터들을 저장하고 검색할 수 있다.
- 내부적으로는 XML 파일로 저장된다.  
  <br>

### SharedPreferences의 한계점

- UI 스레드에서 호출할 수 있도록 API가 설계되었지만, UI 스레드를 블로킹해 ANR을 발생시킬 수 있다.
- 런타임에 예외가 생기면 에러가 발생해 앱이 강제 종료될 수 있다.
- Strong Consistency(강한 일관성)이 보장되지 않아 다중 스레드 환경에서 안전하지 않다.
  > Strong Consistency(강한 일관성)이란 다중 스레드 환경에서 안전하게 데이터를 입력하거나 조회하게 하는 것이다. 이를 통해 다중 스레드 환경에서 안전하게 데이터를 입력, 조회할 수 있도록 한다. 이는 동시성 프로그래밍에서 중요한 요소이다.
- Type Safey가 보장되지 않아 어떤 데이터가 저장되고 추출되는지 일일히 데이터로 Type Convertind(형 변환) 해주어야 한다.

📌 공식문서에서 만약 SharedPrefereces를 사용하고 있다면 DataStore로 이전할 것을 권고하고 있다.

## DataStore

DataStore는 Coroutine을 사용해 동시성 프로그래밍에 최적화된 API를 제공한다.

- 경량 스레드 모델을 구현하는 Coroutine을 사용해 내부를 구현함으로써 더욱 효율적으로 데이터를 저장할 수 있도록 한다.
- 기존 UI 스레드에서 호출되어 ANR을 발생시킬 수 있었던 SharedPreferences와 다르다.
- 내부에서 Coroutine의 IO Dispathcer를 사용해 IO를 담당하는 스레드 풀에서 데이터를 조작하도록 강제했다.
- 또한 Flow를 사용해 데이터를 추출할 수 있도록 만들어 데이터의 입출력을 모두 Coroutine에서 사용할 수 있도록 하였다.  
  <br>

Strong Consistency(강한 일관성)을 보장하는 Transaction API를 제공한다.
<br>  
DataStore는 Type Safety를 지원한다.

- Type Safety란 데이터가 타입을 기준으로 입력되고 출력될 수 있는지에 대한 것이다.
- 즉 “예측불가능한 결과를 내지 않는다”라는 것을 뜻한다.
- 하지만 Preference DataStore, Proto DataStore 중 ProtoDataStore 만 Type Safety를 보장한다.

<div align="center">
  <img alt="SharedPreferences, DataStore 차이" src="https://github.com/DaeYoungee/DaeYoungee.github.io/assets/121485300/4f44a2bf-c45e-4106-b2f3-46ae3d6d0886">   
</div>

## DataStore 구현

DataStore를 사용하여 사용자의 "토큰"을 저장하려고 한다. DataStore는 사용자의 액세스 토큰을 저장하는 좋은 방법이므로 이를 예로 사용. 먼저 필요한 종속성을 설치한다.

### Gradle 라이브러리 추가

```kotlin
implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.4.1'
implementation 'androidx.datastore:datastore-preferences-rxjava2:1.0.0'
implementation 'androidx.datastore:datastore-preferences-rxjava3:1.0.0'
```

### UserStore.kt

data 패키지 밑에 `UserStore.kt`라는 새 파일을 만든다.

```kotlin
import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

class UserStore(private val context: Context) {
    companion object {
        private val Context.dataStore: DataStore<Preferences> by preferencesDataStore("userToken");
        private val USER_TOKEN_KEY = stringPreferencesKey("user_token")
    }

    val getAccessToken: Flow<String> = context.dataStore.data.map { preferences ->
        preferences[USER_TOKEN_KEY] ?: ""
    }

    suspend fun saveToken(token: String) {
        context.dataStore.edit { preferences ->
            preferences[USER_TOKEN_KEY] = token
        }
    }
}
```

DataStore를 관리하기 위한 클래스를 생성한다.

### MainActivity.kt

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import com.example.grade3_2.ui.theme.Grade3_2Theme

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            Grade3_2Theme {
                MainScreen()
            }
        }
    }
}
```

### MainScreen.kt(컴포저블)

```kotlin
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.material.Button
import androidx.compose.material.Text
import androidx.compose.material.TextField
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalSoftwareKeyboardController
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.TextFieldValue
import androidx.compose.ui.unit.dp
import com.example.grade3_2.data.UserStore
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch

@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun MainScreen() {
    val context = LocalContext.current
    val keyboardController = LocalSoftwareKeyboardController.current
    val tokenValue = remember {
        mutableStateOf(TextFieldValue())
    }
    val store = UserStore(context)
    val tokenText = store.getAccessToken.collectAsState(initial = "")

    Column(
        modifier = Modifier
            .fillMaxSize()
            .clickable { keyboardController?.hide() },
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Spacer(modifier = Modifier.height(30.dp))

        Text(text = "DataStorage Example", fontWeight = FontWeight.Bold)

        Spacer(modifier = Modifier.height(15.dp))

        Text(text = tokenText.value)

        Spacer(modifier = Modifier.height(15.dp))

        TextField(
            value = tokenValue.value,
            onValueChange = { tokenValue.value = it },
        )

        Spacer(modifier = Modifier.height(30.dp))

        Button(
            onClick = {
                CoroutineScope(Dispatchers.IO).launch {
                    store.saveToken(tokenValue.value.text)
                }
            }
        ) {
            Text(text = "Update Token")
        }
    }
}
```
