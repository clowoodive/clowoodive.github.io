---
title: "[MySQL] BIT(1) vs TINYINT(1) vs BOOL/BOOLEAN"
excerpt: ""

categories:
    - MySQL
tags:
    - Database
    - MySQL
last_modified_at: 2022-05-26T00:00:00
---

{% capture notice-env %}
#### Environment
- MySQL 5.7
{% endcapture %}

<div class="notice--primary">{{ notice-env | markdownify }}</div>

MySQL에서 BOOL 값을 저장하기 위해 지금 까지 TINYINT(1)을 써왔다. BIT(1)도 1byte 공간을 차지한다고 알고 있었기에 그리했던 것인데 간단히 정리를 해놓기로.

- [BIT[(***M***)]](https://dev.mysql.com/doc/refman/5.7/en/bit-type.html)
    
    A bit-value type. ***`M`*** indicates the number of bits per value, from 1 to 64. The default is 1 if ***`M`*** is omitted.
    
- [TINYINT[(***M***)] [UNSIGNED] [ZEROFILL]](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)
    
    A very small integer. The signed range is `-128` to `127`. The unsigned range is `0` to `255`.
    
- [BOOL](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)`, `[BOOLEAN](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html)
    
    These types are synonyms for [TINYINT(1)](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html). A value of zero is considered false. Nonzero values are considered true:
    

공식 문서의 내용이다. BOOL/BOOLEAN은 TINYINT(1)과 동의어라고 명확하게 적혀 있고, 아래에서 볼 수 있듯이 TINYINT(1)과 BIT(1)은 같은 공간을 차지 한다는 것을 알 수 있다. 

![mysql-bit-tinyint-bool1]({{ '/assets/images/mysql-bit-tinyint-bool1.png' | relative_url }}){: .align-center}

현재 지식으로는 퍼포먼스차이가 있다는데 이해하지 못했고 크게 무게를 두는것 같지는 않다. 팀 규칙으로 BIT(1)을 사용하자고 정한 것은 아니니 TINYINT(1)로 계속 쓸 예정이다. 굳이 TINYINT(1)을 사용하는 이유를 더 꼽아보자면

- [TRUE/FALSE로 정의된 값이지만 추후 여러 타입을 지원하는 값으로 확장될 때 마이그레이션을 대비.](https://www.bennadel.com/blog/3845-why-i-use-tinyint-columns-instead-of-bit-columns-for-boolean-data-in-a-mysql-application.htm)
- [BIT의 MySQL Workbench 버그.](https://bugs.mysql.com/bug.php?id=79604)

정도가 있을것 같다.

참고로 각 타입의 필드를 가지는 테이블을 생성해서 확인 해 보면 아래와 같다.

```sql
CREATE TABLE tinyint_test (
	`bit1` bit(1) NOT NULL,
	`tinyint1` tinyint(1) NOT NULL,    
  `bool` bool NOT NULL,
  `boolean` boolean NOT NULL
);
```

![mysql-bit-tinyint-bool2]({{ '/assets/images/mysql-bit-tinyint-bool2.png' | relative_url }}){: .align-center}