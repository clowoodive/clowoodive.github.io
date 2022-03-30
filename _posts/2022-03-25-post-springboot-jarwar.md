---
title:  "[Spring Boot/Java] Jar & War"
excerpt: "내장 tomcat을 사용해서 서비스 한다면 Jar파일로 패키징 하더라도 실행 가능하다"

categories:
  - Spring Boot
  - Java
tags:
  - Spring Boot
  - Java
last_modified_at: 2022-03-29T00:00:00
---


## Plain Jar

작성한 코드의 클래스 파일과 리소스만 포함 하며, 어플리케이션 실행에 필요한 모든 의존성을 포함하지 않는다.

- plain jar, standard jar, thin jar 라고 불림.
- java -jar 명령어로 실행 불가능.

<br>

## Executable Jar

의존성을 포함한 어플리케이션 실행에 필요한 모든 파일을 포함해서 빌드한다.

- uber jar라고 불림.
- java -jar 명령어로 실행 가능.

<br>

## Spring Boot의 Executable Jar

Java는 nested Jar(jar파일 내에 포함된 jar)를 로드할 수 있는 어떤 표준 방식도 제공하지 않기에 이 문제를 해결하기 위해 shaded jar를 이용한 uber jar로 패키징 하는 방법이 사용되기도 하며, Spring Boot는 아래와 같은 접근 방식으로 실제로 중첩된 jar 구조의 executable jar를 지원한다. 

```
myapp.jar
+-------------------+-------------------------+
| /BOOT-INF/classes | /BOOT-INF/lib/mylib.jar |
|+-----------------+||+-----------+----------+|
||     A.class      |||  B.class  |  C.class ||
|+-----------------+||+-----------+----------+|
+-------------------+-------------------------+
 ^                    ^           ^
 0063                 3452        3980
```

- 사용되는 핵심 클래스는 org.springframework.boot.loader.jar.JarFile.
- 최초 로드 시 각각의 JarEntry위치는 위와 같이 가장 바깥의 물리적 파일 오프셋에 매핑됨.

<br>

## War

Jar 파일이 라이브러리, 어플리케이션 용도의 패키징이라면 War파일은 Servlet/JSP 컨테이너에 배포 가능한 웹 어플리케이션 패키징이다. 따라서 War 패키징은 WEB-INF/META-INF와 같이 사전 정의된 구조가 요구된다.

## 참고

만약 내장 tomcat을 사용해서 서비스 한다면(외장 서블릿 컨테이너를 사용하거나 JSP를 사용하는것이 아니라면) Jar파일로 패키징 하더라도 실행 가능하다. 

<!--

[https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html#appendix.executable-jar.nested-jars](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html#appendix.executable-jar.nested-jars)

[https://zion830.tistory.com/121](https://zion830.tistory.com/121)

[https://wordbe.tistory.com/entry/Spring-Boot-2-Executable-JAR-스프링-부트-실행](https://wordbe.tistory.com/entry/Spring-Boot-2-Executable-JAR-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-%EC%8B%A4%ED%96%89)

[https://hye0-log.tistory.com/27](https://hye0-log.tistory.com/27)

[https://hye0-log.tistory.com/29](https://hye0-log.tistory.com/29)

-->