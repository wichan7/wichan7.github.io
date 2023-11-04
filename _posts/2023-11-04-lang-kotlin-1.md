---
title: "코틀린에 대해 알아보자"
categories: 
  - Lang
tags:
  - lang
  - kotlin
toc: true
---

## 코틀린이란
[코틀린](https://kotlinlang.org/)은 Intellij로 유명한 Jetbrains사에서 만든 언어이다.  
코틀린의 캐치프레이즈는 `Concise. Cross‑platform. Fun`인데, 여러 플랫폼을 지원하는 간결한 언어 정도로 해석할 수 있다.  


### 여러 플랫폼을 지원하는가?
코틀린 공홈의 개요를 보면 `Kotlin Multiplatform`이 제일 처음 소개된다.  
![kmp](/assets/images/lang/kotlin/kotlin-multiplatform.svg)  

이 중, 웹 개발에서 사용한다는 것이 생소해서 알아봤는데, Kotlin/JS 프레임워크 중 하나인 fritz2에서는 아래와 같이 웹 코드를 작성한다.  
``` kotlin
// fritz2
fun main() {
    render {
        div("my-style-class") {
            h2 {
                +"Hello Peter!"
            }
        }
    }
}
```
(저는 JS 쓰겠습니다.)  

본론으로 돌아와서, Kotlin Multiplatform에 의해 한가지의 언어로, 여러 플랫폼 서비스를 달성할 수 있음을 알았다.  
실제 프로젝트에서 많이 사용되는 플랫폼은 Android/iOS, Server(Spring)정도로 보인다.  

### 간결한가?
뒤에서 소개하겠지만, Kotlin은 최종적으로 바이트코드로 변환되어 JVM에 의해 실행되기에 Java 코드와 많이 비교한다.  
``` java
// java
function main() {
    String name = "stranger";
    System.out.println("Hi, " + name + "!");
    System.out.print("Current count:");
    for (int i=0; i<=10; i++) {
        System.out.print(" " + Integer.toString(i));
    }
}
```

``` kotlin
// kotlin
fun main() {
    val name = "stranger"
    println("Hi, $name!")
    print("Current count:")
    for (i in 0..10) {
        print(" $i")
    }
}
```

System.out 등의 패키지명부터 시작했던 Java 표준 라이브러리에 비해 간결함을 볼 수 있다.  

## 코틀린의 빌드 과정
![kmp](/assets/images/lang/kotlin/complie-process.png)  
JVM은 `*.class(바이트코드)`를 실행시키기 위한 가상 머신이다.  
Java는 `Java Compiler`에 의해 *.class 파일로 변환된다.  

Kotlin도 JVM을 사용한다. Java와 마찬가지로 `Kotlin Compiler`에 의해 *.class로 변환된다.  
이 *.class는 JVM의 class loader에 의해 kotlin runtime과 함께 로딩된 후, 각 운영체제에 맞는 네이티브 코드로 변환되어 최종적으로 프로그램이 실행된다.  

## 사족
1. 다음 포스팅에서는 Kotlin Complier를 설치하고 사용해보자.  
2. Kotlin의 원어민스러운 발음은 '캇린' 이라고 한다.  