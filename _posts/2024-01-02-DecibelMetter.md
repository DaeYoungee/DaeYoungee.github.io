---
title: "[Kotlin] DecibelMetter"
excerpt: "코틀린으로 녹음 도중 데시벨 측정"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Decibel]
sidebar:
  nav: "counts"

data: 2024-01-02
last_modified_at: 2024-01-02

published: true
---

### 데시벨 측정을 위한 간단한 설명

이번 포스팅에서는 데시벨 측정 코드부분만 다룬다. 녹음 기능이 궁금하다면 바로 이전 포스팅(MediaRecord)를 찾아보자.
가장 중요한 코드는 `record.maxAmplitude` 이고 녹음 도중 진폭을 측정하는 메서드이다.

## 코드 설명

Koltin Jetpack Compose에서 상태변화를 이용한 실시간 Decibel 변화를 UI로 그려보자. 당연히 늘 그렇게 MVVM 패턴으로 비즈니스 로직은 ViewModel에 표현했다.

### MainActivity

MainActivity의 `onCreate()` 부분은 이 전 포스팅에서 있고 변경된 부분만 코드로 표시했다.

```kotlin
@Composable
fun MainScreen(
    viewModel: RecordViewModel,
    activity: MainActivity
) {

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = "<데시벨 측정>", fontWeight = FontWeight.Bold, fontSize = 50.sp)
        Spacer(modifier = Modifier.height(24.dp))
        Text(text = buildAnnotatedString {
            withStyle(style = SpanStyle(fontWeight = FontWeight.Bold, fontSize = 40.sp)) {
                append(viewModel.db.value)
            }
            append("db")
        })
        Button(onClick = { viewModel.checkAudioPermission(activity = activity) }) {
            Text(text = "마이크 권한")
        }
        Button(onClick = {
            if (!viewModel.recoding.value) {
                viewModel.startRecoding()
                viewModel.changeRecoding()
                viewModel.meterDb()  // 추가된 코드
            } else {
                viewModel.stopRecording()
                viewModel.changeRecoding()
                viewModel.cancelDb()  // 추가된 코드
            }
        }) { Text(text = if (!viewModel.recoding.value) { "녹음하기" } else { "중지하기" }) }
        Button(onClick = {
            if (!viewModel.playing.value) {
                viewModel.playAudio()
                viewModel.changePlaying()
            } else {
                viewModel.stopAudio()
                viewModel.changePlaying()
            }
        }) { Text(text = if (!viewModel.playing.value) { "재생하기" } else { "중지하기" }) }
    }

}
```

### RecordViewModel

새로 추가한 코드부분만 강조하기 위해 동일한 코드는 ... 으로 변경했다.
`recorder.maxAmplitude` 는 진폭을 알려주는 메서드이다. 참고로 recorder는 `MediaRecorder` 객체라는 사실을 모두 알거라고 믿는다.  
`20 * log10(amplitude.toDouble())` 은 진폭을 데시벨로 바꾸는 코드.  
`amplitude`을 0보다 클 때와 작을 때로 분기를 나눈 이유는 `amplitude` 이 0보다 작을 경우 **-infinity** 가 표시되기 때문이다. 0보다 작은 경우 UI에 "00"을 보이게 했다.  
<br>
**ViewModelScope**를 이용해 비동기로 처리했고 `meterDb()` 메서드 호출 시 코루틴에서 계속 실행되게 만들었다. `cancelDb()` 메서드 호출을 통해 비동기로 처리되는 데시벨 변환 로직이 멈춘다.

```kotlin
@HiltViewModel
class RecordViewModel @Inject constructor(
    @ApplicationContext private val context: Context
) : ViewModel() {

    ...

    private val _db = mutableStateOf("00")
    val db: State<String> = _db

    ...
    private lateinit var audioFileName: String
    private lateinit var job: Job

    ...

    fun meterDb() {
        recorder?.let {
            job = viewModelScope.launch {
                withContext(Dispatchers.IO) {
                    while (_recoding.value) {
                        delay(1000)
                        val amplitude = it.maxAmplitude // 진폭 측정
                        if (amplitude > 0) {
                            //진폭이 0 보다 크면 .. toDoSomething
                            //진폭이 0이하이면 데시벨이 -무한대로 나옵니다.
                            val doubleDb = 20 * log10(amplitude.toDouble()) // from 진폭 to 데시벨
                            _db.value = round(doubleDb).toString() // double타입 데시벨을 반올림해서 String으로 타입 캐스팅
                            Log.d("daeYoung", "decibel: $db")
                        } else {
                            _db.value = "00"
                        }
                    }
                }
            }
        }
    }

    fun cancelDb() {
        _db.value = "00"
        job.cancel()
    }

}
```

### Reference

[MediaRecorder에서 데시벨 측정](https://velog.io/@nagosooo/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EB%85%B9%EC%9D%8C%EB%8D%B0%EC%8B%9C%EB%B2%A8-%EC%B8%A1%EC%A0%95)

### 데모 영상

<iframe height="700" src="https://github.com/kailash09dabhi/OmRecorder/assets/121485300/5781ae83-d289-4081-a0af-0aca73ef9c5c" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
