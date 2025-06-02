---
title: "[MySQL] - AUTO_INCREMENT 컬럼이 포함된 레코드 삽입 시 Lock 메커니즘"
excerpt: ""

categories:
  - MySQL
tags:
  - MySQL
  - Database
last_modified_at: 2025-06-01T00:00:00
---

{% capture notice-env %}
#### Environment
 - MySQL 5.7
 - MySQL 8.0
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


MySQL InnoDB 테이블에서 Auto Increment 필드를 가진 레코드가 삽입될 때 테이블 락이 걸리고 순차적으로 AI 키가 발급된다고 알고 있다. 
이는 모드 0의 동작이며 MySQL 5.6/5.7 에서의 default 모드는 1이고, 8.0 에서는 2로 변경되었다.

먼저 확실히 해야 할 것은 락은 레코드 삽입 구문 실행시에 락을 설정하는 것이며, 모드 무관하게 AI 값을 발급받은 트랜잭션이 롤백되면 해당 값은 소실되고 건너뛰게 된다.

# 1. Traditional Lock Mode(`innodb_autoinc_lock_mode = 0`)

`AUTO_INCREMENT` 를 생성하는 모든 `INSERT` 구문(`INSERT`, `INSERT ... SELECT`, `REPLACE` 등)에 대해 테이블 수준 락을 설정하여 AI 값의 연속성을 보장하지만, `INSERT` 작업이 많을수록 락으로 인해 동시성이 떨어진다. 
먼저 AI 값을 발급 받으면서 삽입 하는 트랜잭션이 커밋 or 롤백되지 않으면, 다른 트랜잭션은 AI 값을 발급 받지도 못하기에 롤백 되지 않는 한 커밋 순서도 보장된다고 봐야한다.

# 2. Consecutive Lock Mode(`innodb_autoinc_lock_mode = 1`)

삽입할 행 수를 미리 알 수 없는 Bulk Inserts**(**`INSERT ... SELECT`, `REPLACE ... SELECT`, `LOAD DATA`)에 대해서는 0모드와 같이 테이블 수준 락을 사용하고, 삽입할 행 수를 미리 알수 있는 Simple Inserts에 대해서는 AI 값을 할당할 때에만 전역 카운터에 가벼운 뮤텍스 잠금을 설정하기 때문에 동시성에 이점이 있다.

두 트랜잭션이 삽입 개수를 알수 있는 Simple Inserts를 각각 동시에 수행할 때에는 뮤텍스 설정후 연속 AI 값을 할당 받기에 서로 값이 교차할 수 없다.

- 트랜잭션 A : 11, 12, 13 / 트랜잭션 B : 8, 9, 10 (가능)
- 트랜잭션 A : 9, 11, 13 / 트랜잭션 B : 8, 10, 12 (불가능)

필요한 AI값들을 할당받은 세션들이 알아서 AI값을 사용하게 되므로, 나중 할당받은 세션의 트랜잭션에서 먼저 커밋하면 조회시 먼저 나타날 수도 있다.

# 3. Interleaved Lock Mode(**`innodb_autoinc_lock_mode = 2`**)

모든 `INSERT` 구문에 테이블 수준의 락을 사용하지 않고 뮤텍스를 사용하여 값을 할당하기에 동시성이 가장 높다. 
Simple Inserts의 경우 모드1과 같기에 AI 값이 교차될 수 없고, AI 값을 할당받은 순서가 아니라 커밋하는 순서로 조회 될수 있다.

삽입 개수를 알 수 없는 Bulk Inserts를 두 트랜잭션이 동시에 수행하면 서로 값이 교차 될 수 있다.

- 트랜잭션 A : 9, 11, 13 / 트랜잭션 B : 8, 10, 12 (가능)

모드 1 설정에 Simple Inserts를 사용하는 것이 주된 환경이 될것이라 예상되는데, 한 트랜잭션이 AI 값을 발급받아 삽입한뒤 그 AI값을 최고 값으로 가정하고 비즈니스 로직에서 처리 하는 시점에 이미 다른 트랜잭션에서 더 큰 AI값을 발급받고 심지어 커밋까지 먼저 할수도 있다는 것을 고려해야 한다.

아래는 `innodb_autoinc_lock_mode` 를 확인하고 변경하는 방법이다.

```sql
-- 확인
SHOW VARIABLES LIKE 'innodb_autoinc_lock_mode';

-- 임시 변경
SET GLOBAL innodb_autoinc_lock_mode = 1; -- runtime 변경

-- 영구 변경(재시작 필요)
[mysqld]
innodb_autoinc_lock_mode = 1
```
