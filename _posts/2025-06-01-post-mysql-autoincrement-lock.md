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
이는 **`innodb_autoinc_lock_mode`** 파라미터값 0의 동작이며 MySQL 5.6/5.7 에서의 default 값은 1이고, 8.0 에서는 2로 변경되었다.

먼저 알아둬야 할 것은 AUTO-INC 테이블 레벨 락은 레코드 삽입 구문 실행시에만 적용되며 트랜잭션 내에서 삽입 구문 이후에 락이 유지되는 것이 아니다. 따라서 삽입 구문 자체가 아주 오래걸리는 것이 아닌 이상 트랜잭션 테스트로 락이 설정된 상태를 확인하기는 어렵다.

그리고 모드 무관하게 AI 값을 발급받은 트랜잭션이 롤백되더라도 해당 값은 소실되고 재사용 할 수 없다.

# 1. Traditional Lock Mode(`innodb_autoinc_lock_mode = 0`)

`AUTO_INCREMENT` 를 생성하는 모든 `INSERT` 구문(`INSERT`, `INSERT ... SELECT`, `REPLACE` 등)에 대해 구문이 끝날 때 까지 AUTO-INC 테이블 레벨 락을 설정하여 단일 삽입 구문 내에서 AI 값의 연속성을 보장하지만, `INSERT` 작업이 많을수록 락으로 인해 동시성이 떨어진다. 

# 2. Consecutive Lock Mode(`innodb_autoinc_lock_mode = 1`)

삽입할 행 수를 미리 알 수 없는 Bulk Inserts**(**`INSERT ... SELECT`, `REPLACE ... SELECT`, `LOAD DATA`)에 대해서는 모드 0과 같이 AUTO-INC 테이블 레벨 락을 사용하고, 삽입할 행 수를 미리 알수 있는 Simple Inserts에 대해서는 구문이 끝날 때 까지가 아닌 AI 값을 할당할 때에만 전역 카운터에 가벼운 뮤텍스 잠금을 설정하기 때문에 동시성에 이점이 있다.

Simple Inserts에서도 AI 값이 교차될 수 없고 연속적이며 한 구문 내에서 AI 값에 gap이 생길수 없다.

여러 레코드를 삽입하는 Simple Inserts 에서 몇몇 레코드의 AI 컬럼에 명시적인 값을 지정하는 Mixed Mode Inserts 에 대해서는 더많은 AI 값을 할당(예약)하고 일부값이 소실 될 수는 있으나 그 외에 의미적으로 모드 0과 같다.  

```sql
-- Mixed Mode Inserts 테스트
INSERT INTO test(userid, dataid) VALUES (100, 1), (null, 2), (110, 3), (null, 4);
-- 쿼리 결과
100, 1
101, 2
110, 3
111, 4
-- case 1 - 111까지 삽입후 모드 0에서 추가 삽입 하면 112부터 발급
INSERT INTO test(userid, dataid) VALUES (null, 11), (null, 22), (null, 33), (null, 44);
112, 11
113, 22
114, 33
115, 44
-- case 2 - 111까지 삽입후 모드 1에서 추가 삽입 하면 113부터 발급
INSERT INTO test(userid, dataid) VALUES (null, 11), (null, 22), (null, 33), (null, 44);
113, 11
114, 22
115, 33
116, 44
```

# 3. Interleaved Lock Mode(**`innodb_autoinc_lock_mode = 2`**)

모든 `INSERT` 구문에 테이블 수준의 락을 사용하지 않고 뮤텍스를 사용하여 값을 할당하기에 동시성이 가장 높다. 
Simple Inserts의 경우 모드1과 같기에 AI 값이 교차될 수 없고 한 구문 내에서 AI 값에 gap이 생길수 없지만(Mixed Mode Inserts도 모드 1과 마찬가지로 예외), Bulk Inserts의 경우에는 구문간의 번호 할당이 교차 될 수 있다.

- 구문 A : 9, 11, 13 / 구문 B : 8, 10, 12

# 그 외 고려사항

모드 1 설정에 Simple Inserts를 사용하는 것이 주된 환경이 될것이라 예상되는데, 한 트랜잭션이 AI 값을 발급받아 삽입한뒤 그 AI값을 최고 값으로 가정하고 비즈니스 로직에서 처리 하는 시점에 이미 다른 트랜잭션에서 더 큰 AI값을 발급받고 심지어 커밋까지 먼저 할수도 있다는 것을 고려해야 한다.

그리고 Bulk Inserts를 제외하더라도 구문 내에서만 연속성을 보장하고 gap이 없는 것이므로, 로직의 트랜잭션 내에서 연속성 보장되며 gap이 없다고 단정지어서는 안된다.

아래는 `innodb_autoinc_lock_mode` 를 확인하고 변경하는 방법이다.

```sql
-- 확인
SHOW VARIABLES LIKE 'innodb_autoinc_lock_mode';

-- 영구 변경(재시작 필요)
[mysqld]
innodb_autoinc_lock_mode = 1
```