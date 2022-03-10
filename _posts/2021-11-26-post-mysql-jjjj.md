---
title:  "MySQL 쿼리 최적화 - Index 종류"
excerpt: "다양한 Index 적용 방식을 알아보자"

categories:
  - MySQL
tags:
  - Database
  - MySQL
last_modified_at: 2021-11-29T15:06:00-05:00
---

MySQL SELECT 쿼리 최적화에는 여러가지 방법들이 있지만  
인덱스의 적용 여부가 체감상 가장 크지 않았나 싶다.  
이 글에 정리한 간단한 인덱스 적용보다 더 고차원의 방안이 필요하다면  
공홈의 Optimization 문서 전체를 꼼꼼히 뒤져 볼 필요가 있다.  
주의 할 점은 같은 인덱스 세팅이어도 테스트 데이터 구성 및 환경 세팅에 따라  
Optimizer가 실행계획을 달리 하므로 실제와 유사하게 잘 세팅해야 한다.  
(MySQL 5.7 버전 innodb 기준임을 참고)

## Prefixes 인덱스(단일 컬럼)
BLOB, TEXT 타입의 컬럼에 대해 앞의 몇 문자만 지정해서 인덱스를 생성 할 수 있다.
``` sql
CREATE INDEX part_of_name ON customer (name(10));
```

## FULLTEXT 인덱스(단일 컬럼)
CHAR, VARCHAR, TEXT 필드에 대한 빠른 텍스트 검색을 위해 사용된다.  
사용하려면 인덱싱 방식등 깊게 알아볼 필요가 있다.

## 다중 컬럼 인덱스
가장 왼쪽 컬럼으로 시작하는 부분집합의 인덱스는 모두 적용된다.  
왼쪽이 가장 높은 인덱스 계층이라고 이해하면 쉽다.  
(깊게는 B-Tree 구조를 알아야 하니 다음 기회에..)
> 인덱스가 multi_index(col1, col2, col3)이라면,  
> (col1), (col1, col2), (col1, col2, col3) 각각의 인덱스가 사용 가능.

## MIN(), MAX() 추출 시 인덱스 적용
바로위에 설명한 내용과 일맥상통하는 내용으로  
WHERE절에 첫번째 인덱스 컬럼(col1)이 상수로 적용되면,  
MIN,MAX 대상 인덱스 컬럼(col2)을 빠르게 찾을 수 있다.
> 다중 컬럼 인덱스인 (col1, col2)이 순서대로 적용된다고 이해하면 쉬움.
<!--
    코드블록 위아래에는 빈줄이 있어야 함!!
-->
``` sql
SELECT MIN(col2), MAX(col2)
FROM tbl_name WHERE col1=10;
```

## 인덱스 병합
한 테이블 내의 인덱스들은 병합되어 적용 가능하다.  
> col1과 col2 컬럼이 각각 인덱스 인 경우 인덱스 병합을 시도한다.
``` sql
SELECT * FROM tbl_name
WHERE col1=val1 AND col2=val2;
```

## Primary Key와 혼합
따로 인덱스를 생성하는 것 외에  
기본적으로 Primary Key는 인덱스로 사용되므로  
PK와 혼합해서 적용될 수 있는 인덱스를 구성하는 방법도 있다.


## 인덱스 확장
PK와 따로 정의된 인덱스는 (따로인덱스), (PK)순서로 확장되어 적용된다.  
따로인덱스가 여러개라면 각각 적용.
튜닝하면서 EXPLAIN결과가 혼동을 준 경우가 많았는데, 이 기능도 한몫 했을듯..
> 참고로 확장 인덱스 적용 여부는 EXPLAIN시 key 필드로 알수 없고, key_len, ref, rows, Extra를 잘 봐야 한다.

``` sql
CREATE TABLE t1 (
  i1 INT NOT NULL DEFAULT 0,
  i2 INT NOT NULL DEFAULT 0,
  d DATE DEFAULT NULL,
  PRIMARY KEY (i1, i2),
  INDEX k_d (d)
) ENGINE = InnoDB;
-- extended index(d, i1, i2)
```

## 커버링 인덱스
쿼리로 얻어올 데이터가 인덱스에 모두 속해 있는 경우  
데이터를 읽지 않고(디스크 I/O를 하지 않고) 인덱스에서 필요한 데이터를 쿼리한다.  
다른 트랜잭션 내에서 변경중인 테이블에 대해서는 트랜잭션이 끝날때까지 커버링 인덱스가 동작하지 않는다.


## 해싱된 필드 인덱스
여러 컬럼으로 이루어진 인덱스 대신 해싱된 값으로 인덱스를 구성하면  
특정한 경우에 더 빠를수도 있다.  
실제로 해싱까지는 아니지만, 비슷한 개념으로 조합된 키를 사용해본 적이 있긴 하다.

``` sql
SELECT * FROM tbl_name
WHERE hash_col=MD5(CONCAT(val1,val2))
AND col1=val1 AND col2=val2;
```


## JOIN 시 인덱스 적용
JOIN 조건 컬럼의 DataType과 Size가 같으면 인덱스가 사용될 수 있다.
> 예외적으로 VARCHAR(10)와 CHAR(10)은 적용 되지만, VARCHAR(10) and CHAR(15)은 안됨.
> 문자열 컬럼의 경우 Char Set도 같아야 함.









