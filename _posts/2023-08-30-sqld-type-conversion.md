---
title: "RDBMS의 묵시적인 형변환 (Feat. SQLD)"
categories: 
  - Certification
tags:
  - sqld
---

## 개요  
`INSERT/UPDATE`문을 사용하면서, Data Type이 맞지 않으면 TO_CHAR, TO_DATE 등 항상 명시적으로 형변환해왔었다.  
SQLD 공부 중, Type이 맞지 않는 데이터의 Insert가 정상적으로 수행되는 것을 보고 신기해 정리한다.  

## 예시  
`employee table`  

속성명|데이터타입|옵션
---|---|---
사원번호|BIGINT|PRIMARY KEY
주소|VARCHAR(100)|NOT NULL

위와 같은 테이블이 있다고 하자.  
``` sql
INSERT INTO employee VALUES('1234567', 5112);
UPDATE employee SET 주소=5113 WHERE 주소=5112;
```
위 두 쿼리는 문제없이 성공한다.  

## 유의사항
* 다만 'wichan'과 같은 문자열을 number type으로 캐스팅할 수는 없다.  
이 때에는 `Incorrect integer value` 에러를 받을 수 있다.  