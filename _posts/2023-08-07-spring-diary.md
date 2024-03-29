---
title: "[기록] Spring Boot, Mariadb, JPA, Thymeleaf"
categories: 
  - Spring Framework
tags:
  - web
  - mariadb
  - jpa
---

## 개요
Spring Boot와 관련해 알아보고 작업한 내용을 기록용으로 작성한다.

## 참조
[JPA, Hibernate의 개념](https://suhwan.dev/2019/02/24/jpa-vs-hibernate-vs-spring-data-jpa/)
[Lombok jar로 설치](https://intheham.tistory.com/105)
## 환경 구성  

### Eclipse EE
[download eclipse 2023-06](https://www.eclipse.org/downloads/packages/installer)  

### JDK
[download jdk 20.0.2](https://www.oracle.com/kr/java/technologies/downloads/)  

### DBeaver
[download dbeaver latest](https://dbeaver.io/)  

## 프로젝트 구성

### 프로젝트 생성  
1. Help > Eclipse Marketplace > Spring4(a.k.a sts4) 설치  
2. Spring Boot Stater Project 생성  
3. spring security, spring web, spring web services, lombok, gradle, jpa 설치  

### gradle 설정
``` gradle
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.2.0-SNAPSHOT'
	id 'io.spring.dependency-management' version '1.1.2'
}

group = 'com.example.webdemo'
version = '0.0.1-SNAPSHOT'

java {
	sourceCompatibility = '20'
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
	maven { url 'https://repo.spring.io/milestone' }
	maven { url 'https://repo.spring.io/snapshot' }
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-web-services'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	runtimeOnly 'org.mariadb.jdbc:mariadb-java-client' // MariaDB
	
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.security:spring-security-test'
}

tasks.named('test') {
	useJUnitPlatform()
}
```  

### application.yml 작성
application.properties를 삭제하고 application.yml을 생성한다.  
``` yaml
spring:
  jpa:
    open-in-view: false
    generate-ddl: true
    show-sql: true
    hibernate:
      ddl-auto: update
  datasource:
    url: jdbc:mariadb://localhost:3306/web
    driver-class-name: org.mariadb.jdbc.Driver
    username: root
    password: 990909
```  

### IDE에 Lombok 설치
1. [lombok official](https://projectlombok.org/download) 다운로드  
2. jar 파일 실행  
3. eclipse 종료 후 install  
4. refresh, rebuild  

## DB 구성

### Maria Image Pull
`docker pull mariadb`  

### Container run
1. MARIADB_ROOT_PASSWORD 환경변수 설정  
2. MARIADB_DATABASE 환경변수 설정  
3. /var/lib/mysql을 마운트  

### 직접 접근
`mysql -u root -p`  

### 간접 접근
`root / {MARIADB_ROOT_PASSWORD} 계정정보로 jdbc:mariadb://localhost:3306/web 접근`  

## 빌드와 배포

### 빌드
1. `cd {project dir}`  
2. `gradlew build`  

### 배포
1. `java -jar /build/libs/{.jar file}`  