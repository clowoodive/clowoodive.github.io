---
title:  "[Spring-MSA] Spring Cloud Config Clinet 구축(2)"
excerpt: ""

categories:
    - Spring
tags:
    - Spring Cloud
    - MSA
last_modified_at: 2022-08-08T00:00:00
---

{% capture notice-env %}
#### Environment
- Java 11
- Spring Boot 2.6.10
- Spring Cloud Dependencies 2021.0.3
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


간단하게 이전에 구축한 config server에서 설정을 가져오는 client를 만들어 본다.

코드는 [여기](https://github.com/clowoodive/pilot/tree/main/pilot-spring-cloud-config-client).

### build.gradle

```groovy
plugins {
	id 'org.springframework.boot' version '2.6.10'
	id 'io.spring.dependency-management' version '1.0.12.RELEASE'
	id 'java'
}

group = 'clowoodive.pilot.cloud'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
	mavenCentral()
}

ext {
	set('springCloudVersion', "2021.0.3")
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.cloud:spring-cloud-starter-config'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}

tasks.named('test') {
	useJUnitPlatform()
}
```

### application.properties

```yaml
#config server 저장소의 설정 파일명 : config-dev.yml
spring.application.name=config
spring.profiles.active=dev
spring.config.import=optional:configserver:http://localhost:8888
```

### Controller

main()이 있는 클래스에 *`@RestController`* 애너테이션을 적용해서 get 요청으로 간단하게 호출해서 확인 한다. 확인 할 속성은 config server로 부터 받아 lang 지역변수에 바인딩 된 pilot.lang 값이다.

```java
@SpringBootApplication
@RestController
public class ConfigClientApplication {
    @Value("${pilot.lang}")
    private String lang;

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }

    @GetMapping(value = "/{profile}/lang", produces = MediaType.TEXT_PLAIN_VALUE)
    public String profileLang(@PathVariable("profile") String profile) {
        return String.format("%s profile lang is %s...\n", profile, lang);
    }
}
```

### (참고) Config Server에서 로드 한 저장소 설정 파일

```yaml
# config-dev.yml
pilot:
  target: dev
  lang: ko
  datapath: ../config/config-dev.json
```

```yaml
# config-prod.yml
pilot:
  target: prod
  lang: en
  datapath: ../config/config-prod.json
```

### 테스트

client를 띄운 후 브라우저로 get 요청을 하면 config server의 dev 프로파일의 pilot.lnag 값을 확인 할 수 있다. client가 구동 되는 시점에 config server로 부터 설정을 받아온다.

- http://localhost:8080/dev/lang

```
dev profile lang is **ko**...
```

<!--

[https://www.baeldung.com/spring-cloud-configuration](https://www.baeldung.com/spring-cloud-configuration)

-->