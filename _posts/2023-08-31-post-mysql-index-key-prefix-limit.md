---
title: "[MySQL] InnoDB index key prefix limit"
excerpt: ""

categories:
  - MySQL
tags:
  - MySQL
  - Database
  - InnoDB
last_modified_at: 2023-08-31T00:00:00
---

{% capture notice-env %}
#### Environment
 - MySQL 5.7
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


# 1. 문제 발생

GCP MySQL 환경 구성 시 문제 없었던 CREATE TABLE 쿼리를 Alibaba Cloud의 MySQL 인스턴스에서  실행 시 아래와 같은 key 길이 초과 에러 발생.

```
ERROR 1071 (42000) at line 838: Specified key was too long; max key length is 767 bytes
```

<br>

<br>

# 2. 배경 지식

## 2.1. 문자열 컬럼에서 index prefix

- 문자열 컬럼으로 index를 적용할 때 `col_name(N)` 구문을 통해 해당 컬럼의 처음 N개의 문자들로 index 구성 가능.
- `BLOB` 이나 `TEXT` 컬럼의 경우는 필수로 prefix를 설정해야 하며, `VARCHAR` 타입은 필수가 아님.
- [https://dev.mysql.com/doc/refman/5.7/en/column-indexes.html](https://dev.mysql.com/doc/refman/5.7/en/column-indexes.html)

## 2.2. innodb_large_prefix

- index key prefix limit의 bytes를 결정하는 System Variables의 값으로, 기본값은 ON으로 3072 bytes이고 OFF인 경우 767 bytes가 됨.
- 여러 컬럼을 조합한 index 더라도 이 제한은 각 컬럼에만 적용됨.
- [https://dev.mysql.com/doc/refman/5.7/en/innodb-limits.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-limits.html)

## 2.3. Row Format

- row가 물리적으로 저장되는 방식으로, `innodb_large_prefix` 를 지원하는 타입은 `DYNAMIC`과 `COMPRESSED` 가 있음.
- MySQL 5.7.9의 기본값은 `DYNAMIC`.
- [https://dev.mysql.com/doc/refman/5.7/en/innodb-row-format.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-row-format.html)

<br>

<br>

# 3. 원인

- GCP MySQL 인스턴스에서는 `innodb_large_prefix` 값이 기본값인 ON이고, Alibaba Cloud에서는 OFF로 세팅되어 있음.
    - 확인 쿼리 : SHOW VARIABLES LIKE 'innodb_large%';
- 생성할 테이블에서 인덱스 컬럼의 타입이 VARCHAR(255)인데, utf8mb4(4byte) charset을 사용하고 prefix 설정을 하지 않아서 255 x 4 byte = 1020 bytes로 기본값을 초과함.
- 테스트로 해당 컬럼의 문자 개수를 767 bytes 이하가 되는 191로 변경하면 성공함을 확인.

<br>

<br>

# 4. 해결  방안

- 불필요하게 큰 문자 개수(255)를 필요한 만큼 축소하는 방법. → 같은 세팅으로 다수의 서비스 진행 중이어서 관리 사유로 제외.
- prefix를 설정하는 방법. → 해당 컬럼을 prefix로 끊으면 unique 보장 못함.
- Row Format은 두 클라우드에서 모두 default인 `DYNAMIC` 으로 세팅되어 있기에, `innodb_large_prefix` 값을 변경 가능하므로 ON으로 변경.
- 참고로 `innodb_large_prefix`  값은 MySQL 8.0에서 deprecated 되었으므로, 이로 인한 클라우드에서의 파라미터 세팅이 잘 동작하지 않을 수 있음.(i.e. Alibaba)
- [https://dev.mysql.com/doc/refman/5.7/en/create-index.html](https://dev.mysql.com/doc/refman/5.7/en/create-index.html)
    
    ```
    Prefix support and lengths of prefixes (where supported) are storage engine dependent. 
    For example, a prefix can be up to 767 bytes long for InnoDB tables or 3072 bytes 
    if the innodb_large_prefix option is enabled
    ```
    

<br>

<br>

# 5. 정리

관리 편의 및 `innodb_large_prefix` 의 기본값이 ON 이기에 ON 으로 값을 변경하는 것으로 결정했으며, 해당 컬럼의 문자 개수를 필요 이상으로 크게(255) 세팅 한 것 또한 일괄적인 세팅으로 개발/관리를 편하게 하기 위한 것이지만 앞으로는 지양 해야 할 것으로 생각됨.