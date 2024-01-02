---
title: "[Kotlin] Notification"
excerpt: "Android에서 알림을 띄어보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Notification]
sidebar:
  nav: "counts"

data: 2023-09-12
last_modified_at: 2023-09-12

published: true
---

**Manifest 권한 설정**

```kotlin
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

❗️ 주의: 만약 알림이 울리지 않으면 앱의 설정으로 들어가서 알림기능이 꺼져있는지 확인해보길 바란다.

## 알림 만드는 과정

1. NotificationManager 객체 생성
2. 채널ID 및 채널이름이 등록된 NotificationChannel 객체 생성
3. NotificationChannel의 기타 (소리, 진동, 불빛 등) 특성 설정
4. NotificationManager에 NotificationChannel 등록
5. Notification 객체를 만들기 위해 먼저 NotificationCompat.Builder 객체 생성(매개변수로 채널ID 입력)
6. Builder에 실제 알림 메시지, 아이콘 등을 설정
7. build를 통해 Notification 객체 생성
8. NotificationManager.notify()로 알림 이벤트설정, 매개변수에 채널ID와 Notification 객체 전달

## 알림 채널

`NotificationManager`에 알림 채널을 등록시키기 위해서 `NotificationChannel(채널ID, 채널이름, 중요도)`을 만든다.  
중요도에는  
NotificationManager.IMPORTANCE_HIGH : 긴급 상황으로 알림음 울리며 헤드업으로 표시  
NotificationManager.IMPORTANCE_DEFAULT : 높은 중요도이며 알림음이 울림  
NotificationManager.IMPORTANCE_LOW : 중간 중요도이며 알림음이 울리지 않음  
NotificationManager.IMPORTANCE_MIN : 낮은 중요도이며 알림음 없고 상태 바에도 표시 안함. 등이 있다.

`NotificationCompat.Builder`를 만들 때 `Builder(context, channelId)`과 같은 생성자로 생성한다. `NotificationChannel`은 앱의 알림의 종류를 구분하기 위한 것이다. 만약 알림 채널에 대한 구분이 없는 상태에서 유저가 알림을 끄면 중요한 알림들(배터리 관련)까지 모두 유저에게 전달이 되지 않는다. 하지만 API 26부터는 앱별로 알림을 설정할 수 있음으로 이런 걱정이 없다.

`NotificationChannel` 객체의 알림의 세부정보를 설정할 수 있다.
setDescription(String) : 채널의 설명 문자열  
setShowBadge(Boolean) : 홈 화면의 아이콘에 배지 아이콘 출력 여부  
setSound(sound: Uri, audioAttributes: AudioAttributes) : 알림음 재생  
enableLight(Boolean) 불빛 표시 여부  
setLightColor(argb: Int) : 불빛을 표시한다면 불빛의 생삭  
enableVibration(vibration: Boolean) : 진동 여부  
setVibrationPatter(LongArray!) : 진동을 울린다면 진동의 패턴

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channelId = "one-channel"
            val channelName = "My Channel One"
            val channel =
                NotificationChannel(channelId, channelName, NotificationManager.IMPORTANCE_HIGH)

            // 채널 정보 설정
            channel.description = "My Channel One Description"
            channel.setShowBadge(true)
            val uri: Uri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
            val audioAttributes =
                AudioAttributes.Builder()
                    .setContentType(AudioAttributes.CONTENT_TYPE_SONIFICATION)
                    .setUsage(AudioAttributes.USAGE_ALARM).build()

            channel.setSound(uri, audioAttributes)
            channel.enableLights(true)
            channel.lightColor = Color.RED
            channel.enableVibration(true)
            channel.vibrationPattern = longArrayOf(100, 200, 100, 200)

            // NotificationManager에 채널 등록
            manager.createNotificationChannel(channel)

            // 채널 이용해 빌더 생성
            builder = NotificationCompat.Builder(this, channelId)
            Log.d("daeYoung","버전확인")
        } else {
            builder = NotificationCompat.Builder(this)
        }
        builder.setAutoCancel(true)  // 클릭 시 알림 제거
        builder.setSmallIcon(android.R.drawable.ic_notification_overlay)  //스몰 아이콘
        builder.setWhen(System.currentTimeMillis()) //알림 시각
        builder.setContentTitle("Notification Title")    //제목
        builder.setContentText("Notification Content")   //내용

        val intent = Intent(this, LoginActivity::class.java)
        val pendingIntent =
            PendingIntent.getActivity(this, 10, intent, PendingIntent.FLAG_IMMUTABLE)
        builder.setContentIntent(pendingIntent)
```

## 알림 객체

먼저 `NotificationCompat.Builder`에 추가적인 정보(아이콘, 알림 시각, 제목, 내용)를 설정한다.  
`NotificationCompat.Builder`에 `build()` 메서드를 통해 `Notificiation` 객체를 만든다.  
`manager.notify(채널ID, Notification)`가 통해 실제 알림을 일으키는 코드이다.

## 알림 터치 이벤트

알림은 앱이 아닌 시스템에서 관리하는 상태 바에 출력되는 정보이므로 앱의 터치 이벤트로 처리불가 하다.  
앱에서 사용자가 알림을 터치했을 때 실행해야 하는 정보를 Notification 객체에 담아 두고, 실제 이벤트가 발생하면 Notification 객체에 등록된 이벤트 처리 내용을 시스템이 실행하는 구조로 처리한다. 이벤트가 발생할 때 컴포넌트를 실행하려면 인텐트에 담아서 시스템에 요청해야한다. 이때 PendingIntent를 이용해서 요청한다.  
getActivity, getService, getBroadcast 등 3가지가 있지만 여기서는 `getActivity(context: Context, requestCode: Int, intent: Intent, flags: Int)`를 사용한다.  
**PendingIntent의 flag종류**
FLAG_CANCEL_CURRENT : 이전 생성되어 있는 것을 생성하기 전에 취소하고 새롭게 생성하는 FLAG

- FLAG_IMMUTABLE : 생성된 인텐트를 변경할 수 없음을 나타내는 FLAG
- FLAG_MUTTABLE : 생성된 인텐트가 변경 가능함을 나타내는 FLAG
- FLAG_ONE_SHOT : 이 Intent는 일회성으로 생성하는 FLAG
- FLAG_NO_CREATE : 생성된 Intent가 아직 존재하지 않는 경우 생성하는 대신 null을 반환하는 FLAG
- FLAG_UPDATE_CURRENT : 생성된 Intent 이미 존재하면 이를 유지하되 추가 데이터를 새 Intent에 있는 것으로 대체하는 FLAG

PendingIntent 생성할 때 flag로 FLAG_IMMUTABLE 또는 FLAG_MUTABLE를 사용하지 않으면 **Targeting S+ (version 31 and above) requires that one of FLAG_IMMUTABLE or FLAG_MUTABLE be specified when creating a PendingIntent.**와 같은 오류가 뜬다.

**Reference**  
https://velog.io/@ywown/%EB%8B%A4%EC%9D%B4%EC%96%BC%EB%A1%9C%EA%B7%B8%EC%99%80-%EC%95%8C%EB%A6%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EA%B8%B0-a8d4k4fn#5-%EC%95%8C%EB%A6%BC-%EB%9D%84%EC%9A%B0%EA%B8%B0  
https://velog.io/@tjeong/Android-%EC%95%8C%EB%A6%BC-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EC%B2%98%EB%A6%AC
https://velog.io/@tjeong/Android-%EC%95%8C%EB%A6%BC-%EB%9D%84%EC%9A%B0%EA%B8%B0  
[PendingIntent flag 오류](https://velog.io/@heetaeheo/Targeting-S-version-31-and-above-requires-that-one-of-FLAGIMMUTABLE-or-FLAGMUTABLE-be-specified-when-creating-a-PendingIntent)
[PendingIntent](https://zibro.tistory.com/2)
