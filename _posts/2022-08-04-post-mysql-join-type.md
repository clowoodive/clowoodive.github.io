---
title: "[MySQL] JOIN 종류"
excerpt: ""

categories:
    - MySQL
tags:
    - Database
    - MySQL
last_modified_at: 2022-08-04T00:00:00
---

{% capture notice-env %}
#### Environment
- MySQL 5.7
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


### 사전 지식

- 조인 조건
    - ON - 두 테이블간 조인 컬럼 컬럼명이 다르거나 조건이 필요한 경우.
        - ex) … ON tb1.user_idx = tb2.account_id)
        - ex) … ON tb1.user_idx % 1000 = tb2.server_idx
    - USING - 두 테이블간 조인 컬럼 컬럼명이 같은 경우.
        - ex) … JOIN USING (user_idx, …)
- LEFT/RIGHT 등의 조인에서 매칭 레코드가 없으면 null로 세팅된 레코드로 매칭.
- LEFT/RIGHT 조인에서 OUTER는 생략 가능.

### INNER JOIN

- 조인 조건에 부합하는 레코드, 즉 교집합 레코드들을 묶어줌.
- 만약 조인 조건(ON/USING)을 사용하지 않는 경우 table1.record_count x table2.record_count 만큼의 레코드 반환.(= CROSS JOIN)
- MySQL에서는 JOIN / CROSS JOIN / INNER JOIN 이 모두 같은 동작을 함.
- 아래 쿼리는 같은 표현임.
    
    ```sql
    … LEFT JOIN (tb1, tb2, tb3) ON …
    … LEFT JOIN (tb1 CROSS JOIN tb2 CROSS JOIN tb3) ON …
    ```
    

### LEFT JOIN

- 조인 조건에 의해 매칭되는 right 테이블의 레코드가 없더라도 right 컬럼들을 null로 세팅해서 left 테이블의 레코드는 온전히 결과 집합으로 만들어 내는 조인.
- LEFT JOIN 후 교집합을 제외한 결과 집합은 null 조건으로 추출.
    
    ```sql
    … FROM tb1 
    LEFT JOIN tb2 ON tb1.col = tb2.col 
    WHERE tb2.col IS null; -- or IS NOT null
    ```
    

### RIHGT JOIN

- LEFT JOIN 과 반대.

### FULL OUTER JOIN

- left와 right 테이블의 레코드들이 서로 조인 조건에 매칭되지 않더라도 null로 세팅해서 결과 집합을 만들어 냄.
- MySQL은 지원하지 않으므로 UNION 사용.
    
    ```sql
    SELECT * FROM tb1
    LEFT JOIN tb2 ON tb1.col = tb2.col
    UNION -- 중복 제거. UNION ALL은 중복 제거 안함.
    SELECT * FROM tb1
    RIGHT JOIN tb2 ON tb1.col = tb2.col;
    ```
    
- null 체크를 이용해서 전체 조인 집합에서 교집합 부분만 제외 가능.
    
    ```sql
    SELECT * FROM tb1
    LEFT JOIN tb2 ON tb1.col = tb2.col WHERE tb2.col IS null
    UNION
    SELECT * FROM tb1
    RIHGT JOIN tb2 ON tb1.col = tb2.col WHERE tb1.col IS null;
    ```
    

### CROSS JOIN

- left 테이블의 레코드 각각에 right 테이블의 모든 레코드를 조인.
- 가능한 모든 조합이 만들어 지기에 Cartesian Product라고 함.

### SELF JOIN

- 자신의 테이블을 참조하여 의미는 같지만 컬럼명이 다른 두 컬럼을 조인.
- ex) 사원과 직속 상사 매칭.
    
    ```sql
    SELECT * FROM employee;
    +----+-----------+-------------+
    | id | name      | superior_id |
    +----+-----------+-------------+
    |  1 | 홍길동    |        NULL |
    |  2 | 한석봉    |           1 |
    |  3 | 강감찬    |           2 |
    +----+-----------+-------------+
    
    SELECT emp.id, emp.name, emp_super.name AS sup_name
    FROM employee emp 
    INNER JOIN employee emp_super 
    ON emp.superior_id = emp_super.id;
    
    +----+-----------+-----------+
    | id | name      | sup_name  |
    +----+-----------+-----------+
    |  2 | 한석봉    | 홍길동    |
    |  3 | 강감찬    | 한석봉    |
    +----+-----------+-----------+
    ```
    

### STRAIGT_JOIN

- JOIN과 같지만 left 테이블을 항상 먼저 읽음.
- 조인 옵티마이저가 최적이 아닌 순서로 처리 할 때 강제하기 위한 용도.

<!--

[https://dev.mysql.com/doc/refman/5.7/en/join.html](https://dev.mysql.com/doc/refman/5.7/en/join.html)

-->