---
title:  "[Spring-MSA] Spring Cloud Config Server 구축(1)"
excerpt: ""

categories:
  - Spring
tags:
  - Spring Cloud
  - MSA
last_modified_at: 2022-05-27T00:00:00
---

{% capture notice-env %}
#### Environment
- Java 11
- Spring Boot 2.6.8
- Spring Cloud Dependencies 2021.0.3
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


### 프로젝트 생성

검색 해보니 의존성을 간단하게 추가하는 케이스들이 많아서 따라 하려 했지만, 단순히 `implementation 'org.springframework.cloud:spring-cloud-config-server'` 만 추가하면 아래와 같이 출력되면서 찾지 못한다. 버전을 지정하면 빌드가 되긴 했으나 다들 버전 지정 없이 하길래 찾다가 시간만 날렸다.

```
Could not find org.springframework.cloud:spring-cloud-config-server:.
Required by:
    project :
```

그냥 [https://start.spring.io/](https://start.spring.io/) 가서 생성하자. 만들어진 프로젝트의 의존성을 확인 해 보면 결국 [공식 문서](https://spring.io/projects/spring-cloud) 대로 해야 하는 것!  많은 블로거들이 `dependencyManagement` 를 빼놓은 dependency만 블로그에 올려놓아서 혼동되니 주의.

![spring_msa_spring_cloud_config_server1]({{ '/assets/images/Spring-MSA-springcloudconfigserver/spring_msa_spring_cloud_config_server1.png' | relative_url }}){: .align-center}

```groovy
plugins {
	id 'org.springframework.boot' version '2.6.8'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'clowoodive.pilot'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
	mavenCentral()
}

ext {
	set('springCloudVersion', "2021.0.3")
}

dependencies {
	implementation 'org.springframework.cloud:spring-cloud-config-server'
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

### @EnableConfigServer 애너테이션 지정

프로젝트를 열어서 @EnableConfigServer  애너테이션 붙여준다. controller 같은건 필요없다.

```java
@SpringBootApplication
@EnableConfigServer
public class SpringcloudconfigserverApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringcloudconfigserverApplication.class, args);
	}
}
```

### application.yml 설정

이번에는 appplication.properties 대신 application.yml로 작성해 봤다.

추후 client 까지 하게되면 포트 겹치니 8888로 변경 해주고 아래와 같이 세팅한다.

```yaml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/clowoodive/pilot
#          basedir: target/config-repo
          default-label: main
          searchPaths: pilot-springcloudconfigserver-repo
```

- uri - config 파일들이 저장될 백엔드는 git, 로컬 파일, vault, jdbc, redis, aws s3, credhub 등을 지원하는데 여기서는 Git repository를 사용함. 하위 폴더까지 입력하면 안됨.
- basedir - VCS 기반 백엔드(git, svn)인 경우 config 파일들을 로컬 시스템의 임시 폴더에 복제하게 되는데, basedir로 경로를 설정 할 경우 프로젝트 내에 해당 경로 폴더 안에 복제된다. 이렇게 하면 intellij에서 바로 확인하기 좋지만 git 폴더가 중첩되어 귀찮을 수 있음. 따라서 처음에 확인만 하고 주석 처리.
- default-label - Git repository의 브랜치명.
- searchPaths - 보통은 config 파일 전용으로 repository를 만들지만, 나는 repository가 늘어나는게 싫어서 기존 repository안에 폴더를 만들어서 넣었기에 해당 폴더 경로를 지정함. 지정 안하면 찾지 못함.

### Repository에 config 파일 세팅

그리고 위 설명처럼 uri 경로의 repository에 searchPaths 경로로 폴더를 만들고 안에 config파일들을 두었다.

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

아래와 같은 호출로 확인 가능하다. propertySources에 내용이 나와야 정상적으로 된것.

- [http://localhost:8888/config/prod](http://localhost:8888/config/prod)
- [http://localhost:8888/config-prod.yml](http://localhost:8888/config-prod.yml)

```json
{
  "name": "config",
  "profiles": [
    "prod"
  ],
  "label": null,
  "version": "8c3ac07655562c40fbc70264f03a081526dfd1d8",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/clowoodive/pilot/file:C:\\Users\\ADONIS~1.HAN\\AppData\\Local\\Temp\\config-repo-138741164453556539\\pilot-springcloudconfigserver-repo\\config-prod.yml",
      "source": {
        "pilot.target": "prod",
        "pilot.lang": "en",
        "pilot.datapath": "../config/config-prod.json"
      }
    }
  ]
}
```

controller가 필요없는 이유는 내부적으로 EnvironmentController로 처리하기 때문이고 그 코드는 아래와 같다.

```java
@GetMapping(path = "/{name}/{profiles:(?!.*\\b\\.(?:ya?ml|properties|json)\\b).*}",
			produces = MediaType.APPLICATION_JSON_VALUE)
	public Environment defaultLabel(@PathVariable String name, @PathVariable String profiles) {
		return getEnvironment(name, profiles, null, false);
	}

	@GetMapping(path = "/{name}/{profiles:(?!.*\\b\\.(?:ya?ml|properties|json)\\b).*}",
			produces = EnvironmentMediaType.V2_JSON)
	public Environment defaultLabelIncludeOrigin(@PathVariable String name, @PathVariable String profiles) {
		return getEnvironment(name, profiles, null, true);
	}

	@GetMapping(path = "/{name}/{profiles}/{label:.*}", produces = MediaType.APPLICATION_JSON_VALUE)
	public Environment labelled(@PathVariable String name, @PathVariable String profiles, @PathVariable String label) {
		return getEnvironment(name, profiles, label, false);
	}

	@GetMapping(path = "/{name}/{profiles}/{label:.*}", produces = EnvironmentMediaType.V2_JSON)
	public Environment labelledIncludeOrigin(@PathVariable String name, @PathVariable String profiles,
			@PathVariable String label) {
		return getEnvironment(name, profiles, label, true);
	}
```

<!--

[https://jaehun2841.github.io/2022/03/10/2022-03-11-spring-cloud-config/#RefreshScope가-선언된-bean을-사용하는-bean도-재생성-되나요](https://jaehun2841.github.io/2022/03/10/2022-03-11-spring-cloud-config/#RefreshScope%EA%B0%80-%EC%84%A0%EC%96%B8%EB%90%9C-bean%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-bean%EB%8F%84-%EC%9E%AC%EC%83%9D%EC%84%B1-%EB%90%98%EB%82%98%EC%9A%94)

[https://otrodevym.tistory.com/entry/spring-boot-설정하기-14-spring-cloud-config-설정-및-테스트-소스](https://otrodevym.tistory.com/entry/spring-boot-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-14-spring-cloud-config-%EC%84%A4%EC%A0%95-%EB%B0%8F-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%86%8C%EC%8A%A4)

-->