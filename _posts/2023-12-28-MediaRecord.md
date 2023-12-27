---
title: "[Kotlin] MediaRecord"
excerpt: "코틀린으로 녹음 및 재생해보기"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Record]
sidebar:
  nav: "counts"

data: 2023-12-28
last_modified_at: 2022-12-28

published: true
---

### 녹음 및 재생을 위한 객체 설명

안드로이드에서 녹음을 위한 객체는 `MediaRecorder`, 녹음된 파일을 재생하기 위한 객체는 `MediaPlayer` 라는 점을 알고가자.

## 코드 설명

### Manifest.xml에 추가

녹음 기능을 사용하기위한 코드를 Mainfest.xml 파일에 등록시키자

```kotlin
 <uses-permission android:name="android.permission.RECORD_AUDIO" />
```

### MainActivity

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    private val viewModel by viewModels<RecordViewModel>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            Grade3_2Theme {
                MainScreen(
                    viewModel = viewModel,
                    activity = this
                )
            }
        }
    }
}

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
        Button(onClick = { viewModel.checkAudioPermission(activity = activity) }) {
            Text(text = "마이크 권한")
        }
        Button(onClick = {
            if (!viewModel.recoding.value) {
                viewModel.startRecoding()
                viewModel.changeRecoding()
            } else {
                viewModel.stopRecording()
                viewModel.changeRecoding()
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

권한 설정 로직을 ViewModel에 함수로 넣어 사용했는데 activity를 매개변수로 넘겨주는 게 맞는 방법인지는 모르겠다.

```kotlin
@HiltViewModel
class RecordViewModel @Inject constructor(
    @ApplicationContext private val context: Context
) : ViewModel() {

    private val _recoding = mutableStateOf(false)
    val recoding: State<Boolean> = _recoding

    private val _playing = mutableStateOf(false)
    val playing: State<Boolean> = _playing

    private val recordPermission = android.Manifest.permission.RECORD_AUDIO
    private var recorder: MediaRecorder? = null
    private var player: MediaPlayer? = null
    private lateinit var audioFileName: String

    fun changeRecoding() {
        _recoding.value = !_recoding.value
    }

    fun changePlaying() {
        _playing.value = !_playing.value
    }

    // 퍼미션 검사 및 요청 코드
    fun checkAudioPermission(activity: Activity): Boolean {
        return if (ActivityCompat.checkSelfPermission(
                context,
                recordPermission
            ) == PackageManager.PERMISSION_GRANTED
        ) {
            true
        } else {
            ActivityCompat.requestPermissions(activity, arrayOf(recordPermission), 0)
            false
        }
    }

    fun startRecoding() {
        val recordPath = context.getExternalFilesDir("/")!!.absolutePath
        val timeStamp: String = SimpleDateFormat("yyyyMMdd_HHmmss").format(Date())
        audioFileName = recordPath + "/" + "RecordExample_" + timeStamp + "_" + "audio.mp4";
        recorder = MediaRecorder().apply {
            setAudioSource((MediaRecorder.AudioSource.MIC))     // 오디오 소스 설정, 대부분 MIC 사용
            setOutputFormat((MediaRecorder.OutputFormat.MPEG_4))// 출력 파일 포맷 설정
            setAudioEncoder(MediaRecorder.AudioEncoder.AAC)     // 인코더 설정
            setOutputFile(audioFileName)                        // 출력 파일 이름 설정

            try {
                prepare()                                       // MediaRecorder 초기하 완료
            } catch (e: IOException) {
                Log.d("daeYoung", "prepare() failed")
            }

            start()                                             // 녹음 시작
            Log.d("daeYoung", "recordFileName: $audioFileName")
        }
    }

    fun stopRecording() {
        recorder?.apply {
            stop()                                              // 녹음 중지
            release()                                           // 작업 완료 시 리소스 확보
        }
        recorder = null
    }

    fun playAudio() {
        player = MediaPlayer().apply {
            setDataSource(audioFileName)
            runCatching { prepare() }.onSuccess {
                start()
            }.onFailure {
                Log.d("daeYoung", "${it.printStackTrace()}")
            }
        }
    }

    fun stopAudio() {
        player?.release()
        player = null
    }

}
```

아래의 사진은 실제 저장되는 경로와 파일 이름이다.

- 파일 경로
    <div align="center">
    <img alt="로그에 찍힌 저장 경로" src="https://github.com/postboxat18/GoogleCloudBucket/assets/121485300/2041186d-d70d-40dd-9afc-29c245e780d9">   
    </div>   
    <br>

- 파일 이름
    <div align="center">
    <img alt="device file explorer에 저장된 파일 이름" src="https://github.com/postboxat18/GoogleCloudBucket/assets/121485300/07777a0b-e946-4bfd-b85b-b33f6dda30ab">   
    </div>   
    <br>

### Reference

https://stickode.tistory.com/65
https://developer.android.com/guide/topics/media/mediarecorder?hl=ko

### 데모 영상

<iframe height="700" src="https://github.com/postboxat18/GoogleCloudBucket/assets/121485300/b3d261ce-bb2c-4895-907c-b22c7f24ca38" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
