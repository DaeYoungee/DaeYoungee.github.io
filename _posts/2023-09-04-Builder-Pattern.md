---
title: "[Design Pattern] Builder Pattern(빌더 패턴)"
excerpt: "빌더 패턴과 코틀린에서 사용하는지 알아보자"

writer: DaeYoungEE
categories:
  - Design Pattern
tags:
  - [Kotlin, Java, Builder Pattern]
sidebar:
  nav: "counts"

data: 2023-09-04
last_modified_at: 2022-09-04

published: true
---

## Builder Pattern 이란?

빌더 패턴은 복잡한 객체의 생성 과정과 표현 방법을 분리하여 다양한 구성의 인스턴스를 만드는 생성 패턴이다.  
백문이 불어일견 이란 말이 있듯 밑의 코드를 살펴 보자.

### Builder Pattern 적용 전

**TestBuilderPattern.java**

```java
public class TestBuilderPattern {
    public static void main(String[] args) {
        Student student1 = new Student("대영", 1);
        Student student2 = new Student("승민", 2, "남자");
        Student student3 = new Student("원빈", 3, "여자", 180);
        Student student4 = new Student("경지");
    }
}

class Student {
    private String name;
    private int studentId;
    private String sex;
    private int height;


    public Student(String name) {
        this.name = name;
    }

    public Student(String name, int studentId) {
        this.name = name;
        this.studentId = studentId;
    }

    public Student(String name, int studentId, String sex) {
        this.name = name;
        this.studentId = studentId;
        this.sex = sex;
    }

    public Student(String name, int studentId, String sex, int height) {
        this.name = name;
        this.studentId = studentId;
        this.sex = sex;
        this.height = height;
    }
}
```

위의 코드에선 다음과 같은 단점이 있다.  
생성자로 객체를 생성할 때 매개변수의 위치를 정확히해야하며 매개변수의 수가 많은 경우 코드가 매우 복잡해질 수 있다.

### Builder Pattern 적용 후

**TestBuilderPattern.java**

```java
public class TestBuilderPattern {
    public static void main(String[] args) {
        Student student = new Student.StudentBuilder("대영").studentId(1).sex("남").height(188).build();
        System.out.println(student);
    }
}

class Student {
    private String name;
    private int studentId;
    private String sex;
    private int height;

    private Student(String name, int studentId, String sex, int height) {
        this.name = name;
        this.studentId = studentId;
        this.sex = sex;
        this.height = height;
    }

    @Override
    public String toString() {
        return "Student{" +
                "이름:" + name +
                ", 학번:" + studentId +
                ", 성별: " + sex +
                ", 키: " + height +
                '}';
    }

    public static class StudentBuilder {
        // 필수 매개변수
        private String name;

        // 선택 매개변수
        private int studentId;
        private String sex;
        private int height;

        public StudentBuilder(String name) {
            this.name = name;
        }
        public StudentBuilder name(String name) {
            this.name = name;
            return this;
        }

        public StudentBuilder studentId(int studentId) {
            this.studentId = studentId;
            return this;
        }

        public StudentBuilder sex(String sex) {
            this.sex = sex;
            return this;
        }

        public StudentBuilder height(int height) {
            this.height = height;
            return this;
        }

        public Student build() {

            return new Student(name, studentId, sex, height);
        }
    }
}
```

빌더 패턴을 적용했을 때 장점은 생성자의 매개변수 위치와 타입을 신경쓰지 않고 코드를 작성할 수 있어 훨씬 간단하다.  
실제 사용(표현)할 `Student`가 따로 있고 `Student` 객체를 생성하기 위한 `StudentBuilder` 클래스가 따로 있다. **객체를 생성하는 과정과 표현방법을 분리한다는 말**을 이제 이해했을 거라고 본다.

## 빌더 패턴 만들 때 주의 사항

1. 표현할 클래스의 생성자는 private로 둔다.
2. 빌더 클래스의 생성자는 public으로 하며, 필수 값들에 대해 생성자의 파라미터로 받는다.
3. 빌더 클래스는 Static class로 만들어야 한다.
   (이유: 표현한 클래스의 객체를 만들기 전에 사용할 수 있어야 되기 떄문이다.)
4. 빌더 클래스의 필드값 설정 메서드의 리턴값은 자신의 주소값을 리턴한다.(체이닝 기법)
5. 빌더 클래스의 build() 메서드를 만들어 최종적으로 생성하고자 하는 표현 클래스를 리턴 값으로 넘긴다.

표현할 클래스 = Student  
빌더 클래스 = StudentBuilder

## 코틀린에서 빌더 패턴을 사용할까?

정답부터 얘기하면 사용하지 않는다. 빌더 패턴을 사용하는 이유는 객체 생성의 복잡한 과정을 줄이기 위해 쓰인다.  
그러나 코틀린에선 매개변수의 위치가 달라도 문제가 되지 않으며, 매개변수의 디폴트 값을 설정해 생성자를 오버로딩하지 않아도 된다.  
이유는 다음과 같다.

### 클래스의 필드명과 함께 객체 생성

**Test.kt**

```kotlin
data class Student(
    private val name: String,
    private val studentId: Int,
    private val sex: String,
    private val height: Int,
)

fun main() {
    val student = Student(name = "대영", studentId = 1, sex = "남", height = 188)
}
```

매개변수명과 함께 객체를 생성하기 때문에 코드가 명확해진다.

### 생성자의 매개변수의 디폴트값 설정

```kotlin
data class Student(
    private val name: String,
    private val studentId: Int,
    private val sex: String = "남",
    private val height: Int,
)

fun main() {
    val student = Student(name = "대영", studentId = 1, height = 188)
}
```

매개변수인 sex의 "남" 이라는 디폴트 값을 설정해서 객체 생성 시 필요한 부분의 변수만 설정할 수 있도록 한다.
이는 생성자 오버로딩의 복잡함을 없애주기에 코틀린에서 빌더패턴을 필요없다.

## Reference

https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EB%B9%8C%EB%8D%94Builder-%ED%8C%A8%ED%84%B4-%EB%81%9D%ED%8C%90%EC%99%95-%EC%A0%95%EB%A6%AC
https://readystory.tistory.com/121  
[코틀린에서 빌더 패턴](https://velog.io/@jkh9615/Kotlin%EA%B3%BC-Builder-%ED%8C%A8%ED%84%B4)
