---
title: "[Kotlin] Activity 간 데이터 전달 "
excerpt: "Activity 전환 시 데이터 전달 방법을 알아보자.(단방향, 양방향)"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, Activity]
sidebar:
  nav: "counts"

data: 2024-06-10
last_modified_at: 2024-06-10

published: true
---

<!-- 영상 첨부,  -->

A activity에서 B activity로 화면 전환 시 데이터 전달을 하거나(단방향)  
A activity -> B activity -> A activity로 다시 돌아오면서 B activity에서 연산처리된 결과 데이터를 A activity에서 필요로 할 때 데이터를 전달하는 방법을 알아보자.(양방향)

필자는 맵(B activity)에서 특정 위치를 검색하고 해당 LatLng(위도, 경도)를 A activity로 전달이 필요한 경우에 사용 했다.

## Activity 간 단방향 통신

단방향 통신을 너무 간단하다. MainActivity에서 PostedActivity로 이동하는 메서드를 살펴보자.

```kotlin
private fun startPostedInfoActivity(postId: Long, board: String) {
    val intent = Intent(this, PostedInfoActivity::class.java).apply {
        this.putExtra("postId", postId)
        this.putExtra("board", board)
    }
    startActivity(intent)
}
```

`Intent.putExtra()`를 사용하여 key 값과 value 값을 매개변수에 넣어주면 된다.

## Activity 간 양방향 통신

맵을 사용하는 MainAcvitiy에서 SearchActivity로 이동 후 검색어의 위도와 경도를 가지고 다시 MainAcvitiy로 돌아오는 코드이다.  
`registerForActivityResult()` 를 사용해서 activity의 화면 이동과 데이터 전달을 할 수 있다.

- MainActivity.kt

  ```koltin
  @AndroidEntryPoint
  class MainActivity : ComponentActivity() {
      val activityForResult = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
              if (result.resultCode == RESULT_OK) {
                  Log.d(TAG, "SearchActivity의 반환 값: ${result.data?.getDoubleArrayExtra("latlng").contentToString()}")
                  result.data?.getDoubleArrayExtra("latlng").let {
                      recordViewModel.searchResultPlace = LatLng(it?.getOrNull(0)!!, it.getOrNull(1)!!)
                  }
              }
          }

      override fun onCreate(savedInstanceState: Bundle?) { ... }

      private fun startSearchActivity() {
          val intent = Intent(this, SearchActivity::class.java)
          activityForResult.launch((intent))
      }
  }
  ```

SearchActivity 종료 시 `setResult()` 메서드를 통해 intent를 등록하고 intent의 상태값을 반환한다.  
상태 값은 RESULT_OK와 RESULT_CANCELED로 안드로이드에 이미 상수로 정의 되어 있다.  
finish() 메서드가 종료되면 SearchActivity가 종료되면서 MainAcvitiy에 데이터가 전달된다.

- SearchActivity.kt

  ```koltin
  @AndroidEntryPoint
  class SearchActivity : ComponentActivity() {

        override fun onCreate(savedInstanceState: Bundle?) {...}

        private fun returnMainActivity(place: DoubleArray) {
            val intent = intent.apply {
                putExtra("latlng", place)
            }
            setResult(RESULT_OK, intent)
            finish()
        }
  }
  ```

## Reference

https://pangseyoung.tistory.com/entry/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%95%A1%ED%8B%B0%EB%B9%84%ED%8B%B0%EA%B0%84-%ED%99%94%EB%A9%B4-%EC%A0%84%ED%99%98-%EB%B0%8F-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A0%84%EB%8B%AC-Intent-Kotlin
