---
title: "[Kotlin] sealed class"
excerpt: "kotlin의 문법인 sealed class를 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, sealed class]
sidebar:
  nav: "counts"

data: 2024-01-23
last_modified_at: 2024-01-23

published: true
---

## sealed class란

추상 클래스로 상속받은 자식의 종류를 컴파일러에게 명시해준다. `when()` 문을 이용할 때 가장 많이 활용된다.
즉, 컴파일러가 sealed class를 상속받은 자식 클래스를 모두 알고 있다는 뜻이다.  
<br>
아래의 Animal sealed class가 있다고 해보자.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/8268657d-6b40-4d59-a396-0334818924d1">   
</div>
<br>

`when()` 문법을 이용해 해당 동물의 이름을 가져오는 함수를 살펴보자. Monkey와 Cat 두 가지 객체인지만 판별가능하게 했을 때 오류가 나는 것을 볼 수 있다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/9ae95181-2d57-4f2a-8d14-b50e9925b1cf">   
</div>
<br>

Dog를 추가해줬을 때 빨간줄이 사라져 오류가 해결되는 것을 볼 수 있다.
컴퍼일러는 sealed class의 자식 클래스가 3가지가 있는 것을 알 고 있기 때문에 오류가 뜨는 것이다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/9a95253b-1568-4f9b-aaee-dcfa4b650866">   
</div>

## 왜 sealed class를 사용할까?

제목과 같은 의문점을 해결하기 위해서 추상클래스(abstract)를 상속 받는 자식 클래스가 있다는 예시를 들어보겠다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/03d75a36-da0d-48f3-bc29-4534d0c43cf7">   
</div>
<br>

sealed class와 같은 코드인데 오류가 나는 것을 볼 수 있다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/524c75cf-ca6a-4cfb-8582-0f886eb9de07">   
</div>
<br>

else 문을 추가하여 오류를 해결할 수 있다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/7576b3f9-fc36-4e73-ac2c-0cb6ace5adfb">   
</div>
<br>

❗️ **만약 when() 블럭에서 Dog에 대한 처리가 없다고 해보자.**  
컴파일에서 오류가 잡히지 않고 있다. 만약, 사용자가 강아지의 이름을 원한다면 우리는 사용자의 요구를 만족시키지 못한다. 잘못된 input으로 착각해 else문을 처리하기 때문이다.

<div align="center">
<img alt="snackbarHostState" src="https://github.com/DaeYoungee/Compose_study/assets/121485300/69f0d650-0f5e-44f4-8920-4add54fa6b80">   
</div>
<br>

> `sealed class` 를 사용하면 컴파일러가 `sealed class`를 상속한 자식 클래스를 모두 명확하게 인지하고 있기 때문에 when() 블럭에서 else 문을 사용하지 않아도 된다. -> 오류를 컴파일시점에서 잡을 수 있다.

## sealed class의 특징

- 추상 클래스로 직접 인스턴스 객체 생성이 불가능하다.
- 같은 패키지에서만 sealed class 상속이 가능하다. 만약 패키지 명이 `package com.example.watersavior` 라면 `package com.example.watersavior`내에서만 sealed class 상속이 가능하다.

## enum class와 공통점, 차이점

sealed class는 상속이 가능한 추상 클래스이지만, enum class는 상속이 불가능하다.  
sealed class는 state 변경이 가능한 추상 클래스이지만, enum class는 state 변경이 불가능하다.

## References

https://kotlinworld.com/165  
[sealed class와 enum class](https://velog.io/@ddyy094/Enum-class%EC%99%80-Sealed-class)
