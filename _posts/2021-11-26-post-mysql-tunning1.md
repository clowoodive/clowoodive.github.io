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
급할때를 위해 정리 해 본다.

### 1. 테이블에 다중 컬럼 인덱스가 존재한다면,
### 가장 왼쪽 컬럼으로 시작하는 부분집합의 인덱스는 모두 적용된다.
> 인덱스가 multi_index(col1, col2, col3)이라면,  
> (col1), (col1, col2), (col1, col2, col3) 각각의 인덱스가 사용 가능.

### 2. JOIN 대상 컬럼의 DataType과 Size가 같으면 인덱스가 사용될 수 있다.
> 예외적으로 VARCHAR(10)와 CHAR(10)은 적용 되지만, VARCHAR(10) and CHAR(15)은 안됨.
> 문자열 컬럼의 경우 Char Set도 같아야 함.
<!--
    코드블록 위아래에는 빈줄이 있어야 함!!
-->
``` sql
SELECT MIN(key_part2),MAX(key_part2)
FROM tbl_name WHERE key_part1=10;
```