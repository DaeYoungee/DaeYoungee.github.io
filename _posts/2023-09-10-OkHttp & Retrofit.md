---
title: "[Kotlin] OkHttp & Retrofit2"
excerpt: "OkHttp와 Retrofit2의 관계를 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Java, OkHttp]
sidebar:
  nav: "counts"

data: 2023-09-10
last_modified_at: 2023-09-10

published: true
---

## OkHttp와 Retrofit

OkHttp와 Retrofit은 둘다 Square(회사)에서 만든 HTTP 통신 라이브러리다. OkHttp를 기반으로 더 효율적이게 HTTP 통신을 하도록 만들어진 게 Retrofit이다. 현재 Retrofit을 기준으로 사용하며 부가적인 기능을 위해 client로 OkHttp 객체를 연결해서 사용하는 편이다.

간단하게 OkHttp와 Retrofit를 비교했을 때 장점을 알아보자

**Retrofit**
Json은 자바나 코틀린에서 바로 사용할 수 있는 데이터 형식이 아니기 때문에 JSON을 데이터클래스로 변환해줄 컨버터를 사용해야 한다. Retrofit 객체를 생성할 때 GSON Converter Factory를 연결했다면, Retrofit은 통신 성공 시 응답 객체의 body를 별도의 변환 과정 없이 바로 사용가능하다.

```kotlin
Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

반면 **OkHttp** 에선 반환받은 JSON 객체를 data클래스로 바로 변환이 안되기 때문에, 별도의 과정을 거쳐야 한다.

```kotlin
client.newCall(request).enqueue(
  object : okhttp3.Callback {
    override fun onFailure(call: okhttp3.Call, e: IOException) {
         ...
    }

    override fun onResponse(call: okhttp3.Call,
          response: okhttp3.Response) {
            response.body()?.let {
               val result = Gson().fromJson(it.string(),
                            Model.Result::class.java)
               useResult(result)
            }
        }
)
```

Retrofit을 사용했을 때 더 많은 장점이 있지만, 여기서 다루지 않겠다.

## Retrofit만 사용하면 되는거 아니야?

케이스마다 다르겠지만, 보통 Retrofit의 client에 OkHttp 객체를 연결시킨다.
OkHttp가 제공하는 기능중 Interceptor와 실시간의 통신이 필요한 웹소켓 통신도 OkHttp를 이용해서 한다.

### Interceptor의 사용법

보통 Header에 Auth를 추가하기 위해 많이 사용한다.

client로 쓰는 OkHttp의 사용법  
[1](https://devgeek.tistory.com/56)  
[2](https://devgeek.tistory.com/57)  
[3](https://velog.io/@heetaeheo/okhttp-)

[OkHttp와 Retorfit의 장단점](https://velog.io/@jeongminji4490/Android-OkHttp-Retrofit)
