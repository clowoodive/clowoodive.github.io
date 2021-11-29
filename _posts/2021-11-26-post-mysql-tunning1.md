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

### 테이블에 다중 컬럼 인덱스가 존재한다면,  
### 가장 왼쪽 컬럼으로 시작하는 부분집합의 인덱스는 모두 적용된다.
> 인덱스가 multi_index(col1, col2, col3)이라면,  
> (col1), (col1, col2), (col1, col2, col3) 각각의 인덱스가 사용 가능.
