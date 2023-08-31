---
title: "SQL의 Case Expression (Feat. SQLD)"
categories: 
  - Certification
tags:
  - sqld
---

## 개요  
SQL에서는 CASE WHEN THEN을 활용해 조건에 따른 데이터 처리를 할 수 있다.  
이 CASE WHEN THEN문을 `Case Expression`이라고 부르며, 이를 또 `Searched case expression`, `Simple case expression` 으로 나눈다.  

## searched와 simple의 차이  
``` sql
# Searched case expression
CASE
  WHEN {expression} THEN {value} ELSE {value} END

# Simple case expression
CASE {variable} 
  WHEN {comparable value} THEN {value} ELSE {value} END
```

## 사용 예  
SELECT 결과를, 조건에 따라 보여주고 싶을 때 사용할 수 있다.  
``` sql
SELECT emp_no, (CASE emp_no
  WHEN 1001 THEN 'Y'
  WHEN 1003 THEN 'Y'
  ELSE 'N' END
  ) AS 'vip_yn'
FROM emp
```

또는, 원하는 Order를 주기 위해 ORDER BY에서도 사용이 가능하다.  
``` sql
SELECT *
FROM emp
ORDER BY ( CASE emp_name
  WHEN '서지애' THEN 1
  WHEN '강위찬' THEN 2
  ELSE 3 END )
```