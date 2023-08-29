---
title: "ThymeLeaf 소스 변경 후 자동 리로드하기 (Feat. devtools)"
categories: 
  - Spring Framework
tags:
  - web
  - develop
  - spring
---

## 개요
Spring Boot로 CRUD를 구현하던 중 Thymeleaf 코드(.html)를 수정했는데, 바로 반영되지 않고 재시작해야 반영된다.  
왜 반영이 바로 안되는걸까?  

## 이유
Thymeleaf는 성능 향상을 위해 랜더링된 페이지를 캐싱해둔다.  
그리고 템플릿이 변경되었더라도 리로드하지 않기 때문에, 템플릿을 변경하더라도 바로 반영되지 않는다.  

## 어떻게 할까?
개발에서는, Thymeleaf의 cache 옵션을 disable하면 된다.  
`application.yml`에 아래 프로퍼티를 추가한다.  
``` yaml
spring:
  thymeleaf:
    cache: false
```

## Devtools 소개
내친김에 Java 소스도 변경을 감지해서 다시 빌드되도록 해보자.  
Springboot는 'devtools'라는 멋진 라이브러리를 제공한다.  

## Devtools 설치
### gradle 의존성 추가
``` gradle
dependencies {
  compileOnly 'org.springframework.boot:spring-boot-devtools'
}
```

### application.yml 수정
``` yaml
spring:
  devtools:
    livereload:
      enabled: true
    restart:
      enabled: true
```

## Eclipse 설정 변경
Eclipse 상단 Toolbar의 `Project > Build Automatically` 를 Enable한다.  