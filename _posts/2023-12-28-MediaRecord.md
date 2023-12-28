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

MVVM을 이용한 실제 코드를 살펴보자 + 서버와 API 통신할 때 사용하기위해 mp3파일(오디오파일)을 multipart 파일로 타입캐스팅 해보자

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

### Multipart 타입 캐스팅

Multipart는 이미지와 같은 Url을 폼 데이터 형식으로 서버와 통신하기 위해 사용되는 타입이다.

> **Multipart**는 HTTP를 통해 File을 Server로 전송하기 위해 사용되는 **Content-type** 입니다. HTTP 프로토콜은 크게 Header와 Body로 구분이 되고 데이터는 Body에 들어가서 전송이 되는데, Body에 들어가는 데이터 타입을 명시해주는게 **Content-type**입니다. 이때 타입으로 지정해주는 형태를 MIME 타입으로 지정해줄 수 있는데, Multipart(=multipart/form-data)는 MIME 타입 중 하나입니다.

아래의 stopRecoding() 메서드안에 Multipart로 변환하는 코드를 추가하였다.

```kotlin
fun stopRecording() {
    recorder?.apply {
        stop()                                              // 녹음 중지
        release()                                           // 작업 완료 시 리소스 확보
    }
    recorder = null
    val file = File(audioFileName)
    val requestBody = file.asRequestBody("audio/mp3".toMediaTypeOrNull())
    val multipart = MultipartBody.Part.createFormData(
        "record",
        file.name,
        requestBody
    )
    Log.d("daeYoung", "fileName: ${file.name} filePath: ${file.absolutePath}")
    Log.d("daeYoung", "requestBody: ${requestBody} content-type: ${requestBody.contentType()}")
    Log.d("daeYoung", "multipart: ${multipart} body: ${multipart.body} body-content-type: ${multipart.body.contentType()} header: ${multipart.headers}")

}
```

<설명>  
`File` 객체를 만들고 생성자 매개변수로 오디오 파일이 저장된 경로를 넣는다. `File` 객체의 `asRequestBody()` 메서드를 통해 RequestBody로 타입 캐스팅을 하고 `MultipartBody.Part.createFormData()`를 통해서 Multipart 타입으로 바꿔준다.

- Log 출력 결과
    <div align="center">
    <img alt="로그에 찍힌 RequestBody, Multipart" src="https://github.com/DaeYoungee/DaeYoungee.github.io/assets/121485300/429bf9b5-1779-435c-8c96-5e6c71a6ef26">   
    </div>   
    <br>

### Reference

https://stickode.tistory.com/65
https://developer.android.com/guide/topics/media/mediarecorder?hl=ko  
https://yuar.tistory.com/entry/%EC%9D%8C%EC%84%B1-%ED%8C%8C%EC%9D%BC-%EC%84%9C%EB%B2%84-%EC%97%85%EB%A1%9C%EB%93%9C%ED%95%98%EA%B8%B0

### 데모 영상

<iframe height="700" src="https://github.com/postboxat18/GoogleCloudBucket/assets/121485300/b3d261ce-bb2c-4895-907c-b22c7f24ca38" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
