---
title: "[Kotlin] SharedPreference"
excerpt: "SharedPreference에 Hilt 적용해보기"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose, Hilt, SharedPreference]
sidebar:
  nav: "counts"

data: 2023-08-15
last_modified_at: 2023-08-15

published: true
---

## SharedPreference 는?

[key, value] 형태로 간단한 데이터를 저장하고 불러올 수 있는 SQLite 이다.

> **SQLite**는 데이터베이스 관리 시스템이다.  
> 서버가 아니라 응용 프로그램에 넣어 사용하는 비교적 가벼운 데이터베이스이다.

안드로이드에는 SQLite가 내장되어 있다. SQLite에 저장된 데이터는 어플이 종료되거나, 디바이스 전원이 꺼지더라도 삭제되지 않는다.

주로 자동로그인 기능을 적용할 떄 사용한다.

### SharedPreference에 저장된 데이터는 어디서 확인하나요?

Device File Explorer -> data -> data -> [해당 패키지명] -> shared_prefs  
(패키지명은 보통 **com.example.~~** 로 되어있다.)
<img width="467" alt="shared_pref" src="https://github.com/DaeYoungee/GitBlog/assets/121485300/29b0616a-d514-46d9-9232-13de265f0456">  
다음과 같은 사진으로 구성되어있다.

## On, Off 가능한 자동 로그인 기능 구현

**SharedPreferenceModule.kt**

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object SharedPreferenceModule {

    private const val prefFileName = "userPref"

    @Singleton
    @Provides
    fun preferenceImpl(@ApplicationContext context: Context): SharedPreferences =
        context.getSharedPreferences(prefFileName, Context.MODE_PRIVATE)
}
```

먼저 `SharedPreferenceModule.kt` 을 만든다. 나중에서 시스템에서 인식해서 종속 항목으로 삽입하기위해 `@Module` 모듈화 해주고 모듈을 설치할 컴포넌트를 지정해준다.
`SharedPreference` 객체를 생성해줄 `@Provides` 함수가 하나만 존재 함으로 object 클래스로 해주는 편이 좋다.

**HiltRepository.kt**

```kotlin
class HiltRepository @Inject constructor(private val sharedPreferences: SharedPreferences) {

    // 사용자 토큰을 불러옴
    fun getUserPref(
        userToken: String = "userToken",
        defaultValue: String = "",
    ): String? =
        sharedPreferences.getString(userToken, defaultValue)

    // 사용자 토큰을 저장
    fun setUserPrefs(value: String) {
        sharedPreferences.edit().putString("userToken", value).apply()
    }

    // 자동 로그인 boolean 값 불러옴
    fun getAutoLoginPref(
        autoLoginState: String = "autoLoginState",
    ): Boolean =
        sharedPreferences.getBoolean(autoLoginState, false)

    // 자동 로그인 설정
    fun setAutoLoginPref(value: Boolean) {
        sharedPreferences.edit().putBoolean("autoLoginState", value).apply()
    }

    fun logout() {
        sharedPreferences.edit().clear().apply()
    }
}
```

**HiltViewModel.kt**

```kotlin
@HiltViewModel
class HiltViewModel @Inject constructor(private val repository: HiltRepository): ViewModel() {

    private val _id = mutableStateOf("")
    val id: State<String> = _id

    private val _password = mutableStateOf("")
    val password: State<String> = _password

    // Switch 컴포저블 상태 값
    private val _autoLoginState = mutableStateOf(false)
    val autoLoginState: State<Boolean> = _autoLoginState

    // Switch 컴포저블 상태 변경 함수
    fun setAutoLoginState() {
        _autoLoginState.value = !_autoLoginState.value
    }

    fun setId(id: String) {
        _id.value = id
    }

    fun setPassword(password: String) {
        _password.value = password
    }

    fun getUserPref(): String? =
        repository.getUserPref()

    fun setUserPrefs(value: String) {
        repository.setUserPrefs(value = value)
    }

    fun getAutoLoginPref(): Boolean =
        repository.getAutoLoginPref()

    fun setAutoLoginPref() {
        repository.setAutoLoginPref(value = autoLoginState.value)
    }

    fun logout() {
        repository.logout()
    }
}
```

**MainActivity.kt**

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    private val viewModel: HiltViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        checkAutoLogin()

        setContent {
            Column(
                modifier = Modifier.fillMaxSize().padding(16.dp),
                verticalArrangement = Arrangement.Center,
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                Text(text = "메인 화면", fontSize = 30.sp, fontWeight = FontWeight.Bold)
                Button(onClick = {
                    viewModel.logout()
                    moveLoginActivity()
                }) {
                    Text(text = "로그아웃")
                }
            }
        }
    }

    private fun checkAutoLogin() {
        val state = intent.getBooleanExtra("currentLoginState", false)
        if (!state) {
            if (!viewModel.getAutoLoginPref()) {
                moveLoginActivity()
                finish()
            }
        }
    }

    private fun moveLoginActivity() {
        startActivity(Intent(this, LoginActivity::class.java))
    }

}
```

`LoginActivity`에서 `MainActivity`로 넘겨주는 intent를 통해 현재 로그인이 되어 `MainActivity`가 유지되어야 한다는 것을 알려주고 `SharedPreference` 를 통해 AutoLoginState을 저장했다.

**LoginActivity.kt**

```kotlin
@AndroidEntryPoint
class LoginActivity : ComponentActivity() {
    private val viewModel: HiltViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Column(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp)
            ) {
                TextField(
                    value = viewModel.id.value,
                    onValueChange = { viewModel.setId(it) },
                    placeholder = { Text(text = "ID") })
                TextField(
                    value = viewModel.password.value,
                    onValueChange = { viewModel.setPassword(it) },
                    placeholder = { Text(text = "PW") })
                Row(modifier = Modifier.padding(top = 8.dp)) {
                    Button(modifier = Modifier.padding(end = 16.dp), onClick = {
                        viewModel.setUserPrefs("1234")
                        viewModel.setAutoLoginPref()
                        moveMainActivity(viewModel.autoLoginState.value)
                    }) {
                        Text(text = "로그인")
                    }
                    Switch(
                        checked = viewModel.autoLoginState.value,
                        onCheckedChange = {
                            viewModel.setAutoLoginState()
                        }
                    )
                }
            }
        }
    }

    private fun moveMainActivity(autoLoginState: Boolean) {
        finish()
        val intent = Intent(this, MainActivity::class.java).also {
            it.putExtra("currentLoginState", true)
        }
        startActivity(intent)
    }
}
```

## 데모 영상

<img src="https://github.com/DaeYoungee/GitBlog/assets/121485300/042a980f-871f-4dbc-922c-3727d368afaf" width="50%" height="100%"/>

### GitHub

[깃허브](https://github.com/DaeYoungee/Kotlin_DeepDive)
