---
title: "[자료구조] String, StringBuffer, StrinfBuilder "
excerpt: "String, StringBuffer, StringBuilder의 차이를 알아보자"

writer: DaeYoungEE
categories:
  - Kotlin
tags:
  - [Kotlin, String, StringBuilder, StringBuffer]
sidebar:
  nav: "counts"

data: 2024-05-31
last_modified_at: 2024-05-31

published: true
---

요즘 프로그래머스 코딩테스트를 하고있다. 같은 문제이지만 나는 String을 사용했고 다른 사람의 StringBuilder를 사용하는 것이 많이 보였다. 차이점이 궁금했고 어떤 타입을 써야 더 좋은 코드가 될지 궁금해서 찾아보게 되었다.

## String

String과 (StringBuilder, StringBuffer)와의 차이점은 immutable(불변)하고 mutable(변함)에 있다.

| 클래스        | 특징      |
| ------------- | --------- |
| String        | immutable |
| StringBuilder | mutable   |
| StringBuffer  | mutable   |

<br>

String이 불변하다고 했는데 기존의 문자열에 + 기호를 이용해서 새로운 문자열을 만들 수 있지 않나 생각이 들 것 이다. 필자 또한 그러한 방식으로 해왔다.

String이 불변한 이유를 이제부터 설명하겠다.

String 객체는 한번 생성하면 할당된 메모리의 공간이 변하지 않는다. 기존에 생성한 String 문자열에 +, concat()를 사용해서 추가로 문자열을 붙여도 기존 문자열에 새로운 문자열을 붙이는 것이 아니라

**새로운 String 객체가 만들어지고** 새롭게 만들어진 문자열을 저장하게 된다. (String 객체는 Heap 메모리 영역에 생성되는데 한번 생생된 객체의 내용을 변화시킬 수 없다, 기존 객체가 제거되면 Java 가비지 컬렉션이 회수한다.)

이러한 immutable한 객체인 String은 문자열 연산이 많은 경우 성능이 좋지 않다.

## StringBuffer, StringBuilder

StringBuffer/ StringBuilder은 String과 다르게 동작한다. 문자열 연산으로 기존 객체의 메모리가 부족한 경우 기존 버퍼의 크기를 늘리며 유연하게 동작한다.  
StringBuffer와 StringBuilder의 메서드는 모두 동일하다. 그렇다면 차이점은 무엇일까?

**StringBuffer**는 각 메서드별로 Synchronized Keyword가 존재하여, 멀티스레드 환경에서도 동기화를 지원.  
**StringBuilder**는 동기화를 지원하지 않는다.

> 단일스레드 환경이라면 StringBuilder를 사용하는 것이 좋습니다. 단일 스레드환경에서 StringBuffer를 사용한다고 문제가 되는 것은 아니지만, 동기화 관련 처리로 인해 StringBuilder에 비해 성능이 좋지 않습니다.

### 정리

String은 짧은 문자열을 더할 경우 사용합니다.  
StringBuffer는 문자열 연산이 많고 멀티 스레드 환경에서 사용
StringBuilder는 문자열 연산이 많고 단일 스레드 환경에서 사용

## Reference

[String, StringBuffer, StringBuilder의 차이](https://12bme.tistory.com/42)
