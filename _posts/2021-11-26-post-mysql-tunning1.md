---
title:  "MySQL 쿼리 최적화 - Index"
excerpt: "Index 사용을 위한 기본 지식"

categories:
  - Blog
tags:
  - MySQL
last_modified_at: 2021-11-29T15:06:00-05:00
---

MySQL 쿼리 최적화에는 여러가지 방법들이 있지만  
인덱스의 적용 여부가 체감상 가장 크지 않았나 싶다.  
MySQL DOCUMENTATION에 잘 나와있고 기본적인 내용이지만  
한번 해석 했었다고 나중에도 매끄럽게 이해되지도 않고  
급할 때 참고하기 위해 정리 해 본다.

## 1. 다중 컬럼 인덱스인 경우
가장 왼쪽 컬럼으로 시작하는 부분집합의 인덱스는 모두 적용된다.  
왼쪽이 가장 높은 인덱스 계층이라고 이해하면 쉽다.  
(깊게는 B-Tree 구조를 알아야 하니 다음 기회에..)
> 인덱스가 multi_index(col1, col2, col3)이라면,  
> (col1), (col1, col2), (col1, col2, col3) 각각의 인덱스가 사용 가능.

## 2. JOIN 시 인덱스 적용
JOIN 조건 컬럼의 DataType과 Size가 같으면 인덱스가 사용될 수 있다.
> 예외적으로 VARCHAR(10)와 CHAR(10)은 적용 되지만, VARCHAR(10) and CHAR(15)은 안됨.
> 문자열 컬럼의 경우 Char Set도 같아야 함.

## 3. MIN(), MAX() 추출 시 인덱스 적용
1번에 설명한 내용과 일맥상통하는 내용으로  
WHERE절에 첫번째 인덱스 컬럼(col1)이 상수로 적용되면, 다음 인덱스 컬럼(col2)을 빠르게 찾을 수 있다.
> 부분집합 인덱스인 (col1, col2)를 사용한다고 이해하면 쉬움.
<!--
    코드블록 위아래에는 빈줄이 있어야 함!!
-->
``` sql
SELECT MIN(col2), MAX(col2)
FROM tbl_name WHERE col1=10;
```