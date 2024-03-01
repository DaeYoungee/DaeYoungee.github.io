---
title: "[Kotlin] Android Storage "
excerpt: "Android에서 저장소를 접근하는 방법과 메서드를 알아보자."

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Compose, Storage]
sidebar:
  nav: "counts"

data: 2024-03-01
last_modified_at: 2024-03-01

published: true
---

안드로이드에서 App이 설치되면 App마다 사용할 수 있는 공간이 있다. 공간은 내부공간과 외부공간으로 나뉘는 데 둘의 차이점을 알아보겠다.  
참고로 실제 내부 저장소, 외부 저장소를 확인하고 싶으면 **Device Explorer**에서 확인하면 된다.  
[<span style="font-size: 20px; color:rgb(77,171,254)">**내부 저장소, 외부 저장소의 경로로 접근하는 메서드를 알고싶으면 클릭**</span>](https://daeyoungee.github.io/kotlin/Use-camera/#filepathsxml-%EC%83%9D%EC%84%B1)

## Internal storage (내부 저장소)

**내부 저장소**도 특정 dir에 file, cache 등의 공간이 생성된다. cache의 경우 Device 내부에 저장공간이 부족한 경우 동의를 구하지 않고 삭제할 수 있는 폴더이다. 삭제되서는 안되는 파일이라면 cache 폴더에 저장하면 안된다.  
<br>
App의 내부 저장소는 해당 앱만 사용이 가능하며 다른 앱은 접근이 불가능 하다. file을 read, write 하는데 Permission 없이 접근이 가능하다.  
<br>
`/data/user/[User number]/[package name]/` 으로 폴더 생성된다.  
<br>
보통 `/data/data` 와 `/data/user/0` 폴더가 존재한다. `/data/user/0`은 `/data/data`의 **Soft Link**라고 한다.  
`/data/user/0`가 없는 경우가 있는데 당황하지말고 `/data/data`와 `/data/user/0`은 같다고 생각하자.

> <span style="font-size: 24px; color: black">**Soft Link란?**</span>  
> Soft Link는 심볼릭 링크라고도 한다. 쉽게 말해서 바로가기라고 생각하면된다. 예를 들어 진짜 파일은 특정한 중요 폴더에 넣어 놓은 뒤에, 바탕 화면에 하나의 파일을 만들어  해당 파일을 가리키도록 만드는 것이다.  
> `/data/user/0`를 수정하거나 삭제해도 `/data/data`에는 영향을 미치지 않는다.

### User number

`/data/user/0`에서 0이 의미하는 것이 궁금한 사람이 있을 수 있다. 여기서 "0"은 사용자를 나타내고, "0"은 장치 소유자인 첫 번째 사용자를 뜻한다. 추가 사용자를 생성하면 각 사용자에 대해 이 숫자가 증가한다.

## External storage (외부 저장소)

외부 저장소도 내부 저장소와 마찬가지로 비슷하다. 하지만 외부저장소는 read/ write 시 Permission이 필요하다. 그리고 모든 앱에서 접근이 가능하기 때문에 private를 보장하지 않는다. 또한, PC에서 이 Storage에 접근이 가능하기 때문에 일시적으로 read/ write가 불가능할 수 있다.

### App 전용 폴더

App 전용으로 생성되는 storage가 있다. External이기 때문에 다른 앱에서도 접근이 가능하지만, 중요한 것은 이 App이 삭제된다면 이 공간도 함께 삭제된다. 따라서 다른 App에서 공유되길 원하는 data는 이 곳에 저장되어서는 안된다.

`/storage/emulated/0/Android/data/[package name]/` 으로 폴더 생성된다.

### 공용 폴더

공용 폴더는 App들이 삭제된다고 해도 저장된 data가 삭제되지 않는다.

`/storage/emulated/0/[Content 종류]` 으로 폴더 생성된다.

3가지의 Content 종류만 소개한다. 이외에도 여러 Content 종류가 존재한다.

- Content 종류

  |              용도               |           설명            |
  | :-----------------------------: | :-----------------------: |
  | Environment.DIRECTORY_PICTURES  |     이미지 파일 폴더      |
  |   Environment.DIRECTORY_DCIM    | 카메라로 촬영한 사진 폴더 |
  | Environment.DIRECTORY_DOWNLOADS |    다운로드 파일 폴더     |

## Permission

read/ write를 하려면 AndroidManifest.xml에 다음 권한을 선언해야 한다. Runtime permission이기 때문에 사용자의 허락을 받아야 한다.

```kotlin
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

## Reference

[User Number](https://android.stackexchange.com/questions/39542/confused-by-the-many-locations-of-the-virtual-sdcard)  
<a href="https://codechacha.com/ko/android-data-storage/">https://codechacha.com/ko/android-data-storage/</a>  
<a href="https://velog.io/@hunjison/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EB%82%B4%EB%B6%80%EC%A0%80%EC%9E%A5%EC%86%8C%EC%99%B8%EB%B6%80%EC%A0%80%EC%9E%A5%EC%86%8C-%EA%B2%BD%EB%A1%9C">https://velog.io/@hunjison/안드로이드-내부저장소외부저장소-경로</a>  
[Content 종류](https://crazykim2.tistory.com/488)
