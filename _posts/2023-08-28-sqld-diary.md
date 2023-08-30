---
title: "SQLD 출제 범위 및 정리"
categories: 
  - Certification
tags:
  - sqld
---

## 데이터 모델링
### 모델링의 정의
현실 세계의 데이터를 약속된 표기법으로 표기하는 과정. DB를 구축하기 위한 분석/설계 과정에 활용한다.  

### 모델링 시 유의점  

구분|설명
---|---
중복|데이터베이스가 여러 장소에 같은 정보를 저장하는 잘못을 하지 않도록 함.  
비유연|데이터의 정의를 데이터의 사용 프로세스와 분리함으로써 데이터 모델링은 데이터 혹은 프로세스의 작은 변화가 애플리케이션과 데이터베이스에 중대한 변화를 일으킬 수 있는 가능성을 줄임.  
비일관|데이터 중복이 없더라도 비일관성은 발생. 데이터 모델링을 할 때 데이터와 데이터간 상호 연관 관계에 대한 명확한 정의는 이러한 위험을 사전에 예방.  

### 모델링의 분류
추상화된 순으로 `개념 모델링 → 논리 모델링 → 물리 모델링`  

### ERD 작성 순서
1. 엔터티를 그린다.  
2. 엔터티를 배치한다.  
3. 엔터티칸 관계를 설정한다.  
4. 관계명을 쓴다.  
5. 관계의 참여도(1:n...)를 쓴다.  
6. 관계의 필수여부를 쓴다.  

### 엔터티
현실 세계의 '개체'로 대치할 수 있는 모델링 용어. 대부분 명사이다.  
[개념 정리 바로가기](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=wlgns098&logNo=30183442624)

`발생 시점에 따른 분류`  
기본/키 엔터티, 중심 엔터티, 행위 엔터티 (기중행)  

`유/무형에 따른 분류`  
유형 엔티티, 개념 엔티티, 사건 엔티티 (유개사)  

### 엔티티의 특징
인스턴스가 2개 이상이며, 속성이 2개 이상이어야 한다.  

### 속성의 특징
* 해당 업무에서 사용하는 이름을 부여한다.  
* 문장식으로 속성명을 작성하지 않는다.  
* 약어를 가급적 사용하지 않는다.  
* 전체 데이터 모델에서 명칭의 유일성을 확보하는 것이 좋다.  

### 스키마
외부 스키마(view) → 개념 스키마(table model...) → 내부 스키마(record)  

## 데이터 모델과 성능
### 성능 모델링
설계 시점부터 성능과 관련한 사항이 모델링에 반영되게 하는 것.  

### 함수 종속
사원번호가 사원명을 결정한다고 가정하자.  
종속성 표현에서는 `사원번호 → 사원명` 으로 표현한다.  

### 정규화
1, 2, 3, 3.5, 4, 5 정규형이 있음.  

구분|설명
---|---
1NF|컬럼이 원자 값인지?  (테이블에 반복되는 패턴의 속성이 등장하는지 확인하자)  
2NF|부분 함수 종속이 있지는 않은지? (PK의 일부가 자신의 일부를 결정하는지 확인하자)  
3NF|이행 함수 종속이 있지는 않은지? (A->B, B->C이면 A->C가 되는 것을 확인하자)  
3.5NF|결정자인데 후보키에 안올라온 경우는 없는지?
4NF|다치종속이 있는지?  
5NF|조인종속이 있는지?  

※ 실제 모델링 시 4, 5NF는 불필요한 경우가 많음.  

### 반정규화
정규화의 반대 과정으로 테이블, 속성, 관계를 합친다.  

### 반정규화 절차
1. 반정규화 대상 조사
2. 다른 방법 유도 검토
3. 반정규화 적용

## DBMS
### DB 트랜잭션 격리성이 낮으면 발생하는 문제  
1. Dirty Read  
이것은 트랜잭션 처리된 작업의 중간 결과를 볼 수 있는 현상을 말한다. 즉, commit되지 않은 정보를 볼 수 있는 현상을 말하는 것으로 Read Uncommitted Isolation Level일 때 발생하는 Read 현상이다.  
2. Non-Repeatable Read  
이것은 한 트랜잭션안에서 같은 쿼리를 두번 실행 했을 때, 다른 값이 나오는 Read 현상을 말하는 것으로, 하나의 트랜잭션안에서 여러 스냅샷이 사용되는 경우를 말한다. Read-Committed 이하의 Isolation Level에서 나오는 현상이다. 이 현상은 특정 데이터에 대한 수정이 발생하여 나타나는 Read현상을 말한다.  
3. Phantom Read  
이것은 한 트랜잭션 안에서 첫번째 쿼리 수행 결과와 두번째 쿼리 수행 결과가 다른것을 나타내는 것인데 외부에 동시에 실행중인 트랜잭션의 Insert 작업에 의해 발생하는 Read현상을 말한다. 즉, 결과 범위에 속하지 않은 레코드가 외부 작업에 의해 있을 수도 있고, 없어질 수도 있다는 것을 뜻한다.  

## DDL
### CREATE
``` sql
# 테이블 생성
CREATE TABLE emp(
  emp_no BIGINT PRIMARY KEY,
  emp_address VARCHAR(100),
  CONSTRAINT fk_address FOREIGN KEY(emp_address) REFERENCES addresses(addr) ON DELETE CASCADE
)

# 인덱스 생성
CREATE INDEX idx_empno ON emp(emp_no ASC);
```

### ALTER  
``` sql
# 컬럼, 제약조건 추가
ALTER TABLE emp ADD COLUMN emp_address varchar(100) NOT NULL;
ALTER TABLE emp ADD CONSTRAINT fk_address FOREIGN KEY(emp_address) REFERENCES addresses(addr);

# 컬럼 변경
ALTER TABLE emp MODIFY COLUMN emp_address varchar(200);

# 컬럼 이름까지 변경
ALTER TABLE emp CHANGE COLUMN emp_address emp_address2 varchar(300);

# 컬럼, 제약조건 삭제
ALTER TABLE emp DROP COLUMN emp_address2;
ALTER TABLE emp DROP CONSTRAINT fk_address;

# 테이블명 변경
ALTER TABLE emp RENAME emp_new;
```

### DROP  
``` sql
DROP TABLE emp
```

### RENAME
``` sql
RENAME emp TO emp_new
```

## DML
### SELECT
### INSERT
### UPDATE
### DELETE

## DCL
### GRANT
### REVOKE

## TCL  
### COMMIT  
### ROLLBACK