---
title: "[Spring Boot] 구동 시 SQL 데이터베이스 초기화(H2, MySQL)"
excerpt: ""

categories:
  - Spring
tags:
  - SQL
  - Database Initialization
  - Startup
last_modified_at: 2023-03-22T00:00:00
---

{% capture notice-env %}
#### Environment
 - Java 11
 - Spring Boot 2.5.12
 - Gradle 7.5.1 
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


Spring Boot는 구동 시점에 초기화를 담당하는 Bean과 이에 의존하는 Bean을 스스로 감지해서 DDL/DML 스크립트 실행을 통해 SQL Database 초기화를 지원한다.

여기에서는 MySQL을 사용했으니 `DataSourceScriptDatabaseInitializer`가 초기화를 담당하고, 그에 의존하는 Bean은 `JdbcTemplate`의 인터페이스인 `JdbcOperations` 가 될 것이다.

기본 세팅에서는 in-memory DB(ex. H2)에만 적용되기에 외부 DB에 초기화를 적용하기 위해서는 디폴트 `spring.sql.init.mode=embedded` 설정 값을 아래와 같이 변경해 주어야 한다.

```coffeescript
spring.sql.init.mode=always
```

<br><br>

# DB 연결 구성(MySQL)

간단하게 자동으로 구성되는 MySQL 연결 정보를 설정한다. 

물론 로컬에 MySQL이 설치되어 있어야 한다.

```coffeescript
# application.properties

spring.datasource.url=jdbc:mysql://127.0.0.1:3306/Investing
spring.datasource.username=ID
spring.datasource.password=PW
```

<br><br>

# DB 연결 구성(in-memory H2)

H2 의존성을 추가하고 다른 연결구성이 없다면 Spring Boot가 자동으로 H2를 구성한다.

```coffeescript
# build.gradle

dependencies {
  ...
	runtimeOnly 'com.h2database:h2'
  ...
}
```

스크립트는 H2에 맞게 작성 해야 하며, H2 DB 내용을 확인하기 위해 아래와 같이 임베디드 GUI콘솔을 활성화 하는것이 좋다.

```coffeescript
# application.properties

spring.h2.console.enabled=true
```

콘솔은 브라우저에서 아래 경로로 접속 할 수 있으며, 설정을 통해 경로를 변경 할 수도 있다. 

- http://127.0.0.1:포트/h2-console

DB 로그인은 아래와 같은 로그로 출력된 db정보를 콘솔에 입력해서 할 수 있다.

```coffeescript
o.s.b.a.h2.H2ConsoleAutoConfiguration : H2 console available at '/h2-console'. Database available at 'jdbc:h2:mem:bc6d1490-924e-4d0b-945b-bbba9fa172ba'
```

<br><br>

# SQL Script 파일 경로 설정

기본적으로 standard root classpath location은 resources 폴더이고, 이 아래에 `schema.sql`과 `data.sql` 스크립트 파일을 두면 된다.

```coffeescript
# folder structure

project
 |
 +-src
 |  +-main
 |     +-resources
 |        +-schema.sql
 |        +-data.sql
```

스크립트 파일의 경로를 변경하고 싶으면 아래와 같이 경로를 지정하면 되는데, 나는 profile을 DB 플랫폼으로 지정했고 그에따라 해당 DB 플랫폼의 폴더 아래의 스크립트를 사용하게 해서 조금 복잡하다.

```coffeescript
# application.properties

# profile for DB : [h2|mysql]
spring.profiles.active=mysql

# DB initialize
database=${spring.profiles.active}
spring.sql.init.schema-locations=classpath*:db/${database}/schema.sql
spring.sql.init.data-locations=classpath*:db/${database}/data.sql
```

변경된 경로의 디렉토리 구조는 아래와 같다.

```coffeescript
# folder structure

project
 |
 +-src
 |  +-main
 |     +-resources
 |        +-db
 |           +-mysql
 |              +-schema.sql
 |              +-data.sql
```

만약 폴더로 경로를 구분하지 않고 간편하게 profile처럼 `schema-${platform}.sql` 와 같은 방식이 적용되게 하려면 아래와 같이 설정하면 된다.

```coffeescript
# application.properties

spring.sql.init.platform=mysql
```

```coffeescript
# folder structure

project
 |
 +-src
 |  +-main
 |     +-resources
 |        +-schema-mysql.sql
 |        +-data-mysql.sql
```

<br><br>

# 초기화 관련 로그 출력

```coffeescript
# application.properties

logging.level.org.springframework.jdbc.datasource.init=DEBUG
```

<br><br>

# Script 실패 시 애플리케이션 구동

DB초기화가 실패하면 애플리케이션 구동일 실패한다. DB초기화가 실패해도 시작되게 하려면 아래와 같이 설정하면 된다.

```coffeescript
# application.properties

spring.sql.init.continue-on-error=true
```

<br><br>

# Script 작성 팁

스크립트는 각 DB 플랫폼에 맞는 언어로 작성하면 되고, 반복적으로 애플리케이션을 실행하면서 DB초기화가 적용되면 DDL/DML 쿼리 시 에러가 발생 할수 있다. 

이때 위에서 설명한 `spring.sql.init.continue-on-error` 옵션을 켜서 넘어가는 방식은 중복 쿼리로 초기화가 실패한 것인지 다른이유 때문인지 판단하기 번거롭다. 따라서 아래와 같이 DDL은 `DROP` 후 생성 하거나 `IF NOT EXISTS` 문을 사용하고, DML은 `IGNORE` 수정자를 사용하는 것이 편한 것 같다.

```sql
# schema.sql

CREATE TABLE IF NOT EXISTS product (
product_id int(11) NOT NULL,
amount bigint(20) NOT NULL,
PRIMARY KEY (product_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

```sql
# data.sql

INSERT IGNORE INTO productVALUES (1, 999999);
```

코드는 [여기](https://github.com/clowoodive/toy/tree/main/investing).

위 프로젝트의 설정 참고.

### References

- [https://madplay.github.io/post/run-code-on-application-startup-in-springboot](https://docs.spring.io/spring-boot/docs/2.5.12/reference/htmlsingle/#howto.data-initialization)
- [https://www.baeldung.com/spring-boot-h2-database](https://www.baeldung.com/spring-boot-h2-database)