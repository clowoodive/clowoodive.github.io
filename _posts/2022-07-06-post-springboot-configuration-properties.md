---
title: "[Spring Boot] @ConfigurationProperties로 구성 속성 바인딩 하는 몇가지 방법"
excerpt: ""

categories:
  - Spring Boot
tags:
  - Configuration
last_modified_at: 2022-07-06T00:00:00
---

{% capture notice-env %}
#### Environment
- Java 11
- Spring Boot 2.5.14
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


외부화된 구성 속성을 POJO에 바인딩 하는 몇가지 방법을 살펴본다.

우선 application.properties 파일은 아래와 같다.

```yaml
my.account.my-id=123
my.account.my-name=clowoodive
my.account.my-ip-address=1.2.3.4
my.account.port=8888
```

먼저 가장 기본적인 방법이다.

```java
@Setter
@Getter
@Configuration
@ConfigurationProperties(prefix = "my.account")
public class FirstConfig {
    private int myId;
    private String myName;
    private InetAddress myIpAddress;
    private int port;
```

표준 setter가 필요하기에 @Setter를 사용하고, @ConfigurationProperties 애너테이션으로 접두어만 지정 해 주면 SpringBootApplication 실행 시 손쉽게 바인딩 된다. 그리고 @Configuration 을 통해 Bean으로 등록된다.

@Configuration을 사용하지 않는 경우 아래와 같이 @EnableConfigurationProperties 를 사용해서 바인딩 해야 한다. 꼭 main application class에만 사용 가능한 것은 아니고 @Configuration이 선언된  클래스에서도 @EnableConfigurationProperties을 사용 할 수 있다.

```java
@Setter
@Getter
//@Configuration
@ConfigurationProperties(prefix = "my.account")
public class SecondConfig {
    private int myId;
    private String myName;
    private InetAddress myIpAddress;
    private int port;
}
```

```java
@SpringBootApplication
@EnableConfigurationProperties(SecondConfig.class)
public class SecondConfigurationApplication {
	public static void main(String[] args) {
		SpringApplication.run(SecondConfigurationApplication.class, args);
	}
}
```

Spring Boot 2.2부터는 @ConfigurationPropertiesScan 를 사용하여 스캔을 할 수 있고, 기본적으로 패키지를 지정하지 않으면 애너테이션을 선언한 패키지 내에서 스캔 한다.

아래와 같이 @Configuration 과 함께 해당 클래스에 선언하는 방법도 있고, 

```java
@Setter
@Getter
@ConfigurationProperties(prefix = "my.account")
@Configuration
@ConfigurationPropertiesScan
public class ThirdConfig {
    private int myId;
    private String myName;
    private InetAddress myIpAddress;
    private int port;
}
```

아래와 같이 main application class에만 선언 해도 동작 한다.

```java
@Setter
@Getter
@ConfigurationProperties(prefix = "my.account")
//@Configuration
//@ConfigurationPropertiesScan
public class FourthConfig {
    private int myId;
    private String myName;
    private InetAddress myIpAddress;
    private int port;
}
```

```java
@SpringBootApplication
@ConfigurationPropertiesScan
//@ConfigurationPropertiesScan("clowoodive.example.boot.springbootconfigurationprofile")
//@ConfigurationPropertiesScan({"clowoodive.example.boot.springbootconfigurationprofile", "clowoodive.example.boot.springbootconfigurationprofile.config"})
public class FourthConfigurationApplication {

	public static void main(String[] args) {
		SpringApplication.run(FourthConfigurationApplication.class, args);
	}
}
```

코드는 [여기](https://github.com/clowoodive/example/tree/main/example-spring-boot-configuration-properties)를 참고.

<!--

[https://www.baeldung.com/configuration-properties-in-spring-boot](https://www.baeldung.com/configuration-properties-in-spring-boot)

-->