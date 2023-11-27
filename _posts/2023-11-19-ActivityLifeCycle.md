---
title: "[Kotlin] MVVM Pattern"
excerpt: "안드로이드 디자인 패턴, MVVM 패턴에 대해 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin]
sidebar:
  nav: "counts"

data: 2023-11-19
last_modified_at: 2022-11-19

published: true
---

## MVVM 패턴

MVVM 패턴이란 Model, View, ViewModel을 가리키며 ViewModel을 사용하여 Model과 View를 분리하는 것이 특징이다.

<div align="center">
  <img alt="MVVM 패턴 흐름" src="https://github.com/DaeYoungee/DaeYoungee.github.io/assets/121485300/99e20c1f-bf70-474a-b260-76d5ed70565b">   
</div>

### View - 뷰

UI Controller를 담당하는 Acticity, Fragment 이다.
화면에 무엇을 그릴지 결정하고 사용자와 상호작용한다. 데이터 변화를 감지하기 위한 옵저버를 가지고 있다.(보통은 LiveData를 구독하는 형태 -> 참고로 5년전)

### ViewModel - 뷰모델

뷰모델은 UI를 위한 데이터를 가지고 있다. 구성(configuration)이 변경되어도 살아남는다. ex) 화면 회전, 언어 변경 , 테마 변경, 화면 크기 변경, 줌 레벨, 폰트 변경  
뷰모델은 뷰와 분리 되어 있기 때문에 뷰가 Destroy되었다가 Create되어도 뷰모델은 종료되지 않은 데이터를 가지고 있다.

### Repository - 레포지토리

뷰모델과 상호작용하기 위해 데이터 API를 들고 있는 클래스이다. 앱에 필요한 데이터를(내장DB or 외장DB)를 가져온다.  
뷰모델은 DB나 서버에 직접 접근하지 않고, 레포지토리에 접근하는 것으로 앱의 데이터를 관리한다.

> 📌 뷰모델을 왜 사용할까?  
> 뷰모델은 액티비티보다 생명주기가 길어 구성 변경이 잃어나도 데이터를 보존할 수 있고 프레젠테이션 영역과 비즈니스 로직 영역을 나눠 보다 깔끔한 코드를 짤 수 있고 유지보수가 좋기 떄문이다.
>
> > View와 View에 대한 로직을 분리하고, ViewModel을 UnitTest 하기 위한 목적이다.

즉, 정리하자면 <span style="color:rgb(77,171,254)">**View**</span>는 사용자에게 화면을 보여주기만 하는 역할이고 ViewModel의 데이터를 Observing하고 있다가 데이터의 값이 변경되면 화면을 렌더링(컴포징)해준다.  
<span style="color:rgb(77,171,254)">**ViewModel**</span>은 UI(View)와 관련된 데이터를 관리하는 **비즈니스 로직**을 처리한다.  
내장DB(room) or 외장DB(API 통신)과 관련된 데이터는
<span style="color:rgb(77,171,254)">**Repository**</span>를 통해 관리하고 Repository를 통해 받은 데이터를 ViewModel에 저장하고 UI를 렌더링할 때 쓰인다.  
<span style="color:rgb(77,171,254)">**Model**</span>의 대표적인 예시로 Retrofit 이나 Room이 있고 sql 쿼리문으로 데이터를 직접 CRUD할 수 있다. 즉, 데이터를 다루는 메서드를 정의하는데 사용된다.

## MVVM의 안티 패턴

MVVM에서 잘못된 코드는 Context, Application을 주입받는 행동이다.

Context나 Application을 주입받으면 할 수 있는 일

- Context를 Activity 형 변환 할 수 있다.
- Context를 이용한 권한 획득 요청할 수 있다.
- Context를 이용한 Resource에 접근할 수 있다.
- Context를 이용한 다른 화면으로의 전환도 가능하다.

### Context를 주입할 경우

Context를 매핑해 객체를 생성하여, ViewModel에 주입하는 경우 문제가 발생할 수 있다.  
Context는 Activity가 가지고 있다. configuration change가 발생하면 Context가 새로 발행된다. 그말은 ViewModel은 유지된 상태인데, Context는 바뀌었다는 점이다.

Activity에서 Context를 ViewModel에 주입했을 때, 앱은 살아있고 Activity가 종료된다면 ViewModel은 비어있는 Context를 참고하기 때문에 NullpointException이 일어난다.

그래서 Application을 가져다 쓰는 경우도 있는데, Theme 정보가 Activity에서 만 다르다면 Resource를 불러오는 부분에서 문제가 생길 수 있겠다.

### 피드백

뷰는 뷰모델을 알아야되지만, 뷰모델은 뷰를 절대 알면 안된다.

## Reference

[MVVM 패턴 개념](https://aonee.tistory.com/48)  
[MVVM 안티 패턴](https://thdev.tech/android/2023/01/27/Android-Follow-MVVM-03/)
