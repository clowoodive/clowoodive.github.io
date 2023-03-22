---
title:  "[Spring Boot/Java] Jar & War"
excerpt: "내장 tomcat을 사용해서 서비스 한다면 Jar파일로 패키징 하더라도 실행 가능하다"

categories:
  - Spring
  - Java
tags:
  - Spring Boot
  - Java
  - jar
  - war
last_modified_at: 2022-06-30T00:00:00
---


## Plain Jar

작성한 코드의 클래스 파일과 리소스만 포함 하며, 어플리케이션 실행에 필요한 모든 의존성을 포함하지 않음.

- plain jar, standard jar, thin jar 라고 불림.
- java -jar 명령어로 실행 불가능.

## Executable Jar

의존성을 포함한 어플리케이션 실행에 필요한 모든 파일을 포함해서 빌드.

- uber jar 또는 fat jar라고 불림.
- java -jar 명령어로 실행 가능.
- Spring Boot로 패키징 시 내장 tomcat을 포함함.

## Spring Boot의 Executable Jar

Java는 nested Jar(jar파일 내에 포함된 jar)를 로드할 수 있는 어떠한 표준 방식도 제공하지 않기에 이 문제를 해결하기 위해 shaded jar를 이용한 uber jar로 패키징 하는 방법이 사용되기도 하는데, 실제로 어떤 라이브러리가 있는지 확인하기 어렵고 파일명이 같을 경우 문제가 될 수 있다. 

이에 Spring Boot는 다른 접근 방식으로 실제로 중첩된 jar 구조의 executable jar를 지원한다. 

### **Executable Jar File Structure**

```groovy
example.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-BOOT-INF
    +-classes
    |  +-mycompany
    |     +-project
    |        +-YourClasses.class
    +-lib
       +-dependency1.jar
       +-dependency2.jar
```

## War

Jar 파일이 라이브러리, 애플리케이션 용도의 패키징이라면 War파일은 Servlet/JSP 컨테이너에 배포 가능한 웹 애플리케이션 패키징이다. 따라서 WEB-INF/META-INF와 같이 웹 애플리케이션의 사전 정의된 구조로 패키징 된다.

- Spring Boot에서 bootJar로 빌드하면 단독실행 가능.

> Servlet 컨테이너(ex 외장 tomcat)에 구동시키기 위해서 main이 있는 클래스가 SpringBootServletInitializer를 상속받아야 한다.
> 

### **Executable War File Structure**

```groovy
example.war
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-WEB-INF
    +-classes
    |  +-com
    |     +-mycompany
    |        +-project
    |           +-YourClasses.class
    +-lib
    |  +-dependency1.jar
    |  +-dependency2.jar
    +-lib-provided
       +-servlet-api.jar
       +-dependency3.jar
```

### 색인 파일

Spring Boot Loader 호환 jar, war 파일은 `BOOT-INF/` 폴더 아래에 인덱스 파일들을 포함 할 수 있다.

- classpath.idx - nested 의존 jar를 클래스 경로에 추가해야 하는 순서 기록.
- layers.idx - Docker/OCI image 생성을 위해 jar를 논리적 계층으로 분할.(jar에만 가능)

### Spring Boot의 JarFile 클래스

- standard jar나 nested child jar에서 컨텐츠 로드.
- 사용되는 핵심 클래스는 org.springframework.boot.loader.jar.JarFile.
- 최초 로드 시 각각의 JarEntry위치는 아래와 같이 가장 바깥의 물리적 파일 오프셋에 매핑됨.
- 압축을 풀지 않아도 되고, 모든 데이터를 메모리에 올릴 필요 없어짐.

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

## 참고

만약 내장 tomcat을 사용해서 서비스 한다면(외장 서블릿 컨테이너 또는 JSP를 사용하는것이 아니라면)  Jar파일로 패키징 하더라도 실행 가능하다. 

<!--

[https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html#appendix.executable-jar.nested-jars](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html#appendix.executable-jar.nested-jars)

[https://zion830.tistory.com/121](https://zion830.tistory.com/121)

[https://wordbe.tistory.com/entry/Spring-Boot-2-Executable-JAR-스프링-부트-실행](https://wordbe.tistory.com/entry/Spring-Boot-2-Executable-JAR-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-%EC%8B%A4%ED%96%89)

[https://hye0-log.tistory.com/27](https://hye0-log.tistory.com/27)

[https://hye0-log.tistory.com/29](https://hye0-log.tistory.com/29)

-->