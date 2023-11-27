---
title: "[Kotlin] DataStore"
excerpt: "코틀린 제트팩 컴포즈에서 DataStore가 어떻게 사용되는지 알아보자"

writer: DaeYoungEE
categories:
  - kotlin
tags:
  - [Kotlin]
sidebar:
  nav: "counts"

data: 2023-11-27
last_modified_at: 2022-11-27

published: true
---

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
