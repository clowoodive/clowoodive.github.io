---
title: "[Book] Spring in Action - 1 ~ 2장. 웹 애플리케이션 개발하기"
excerpt: ""

categories:
  - Book
tags:
  - Spring in Action
last_modified_at: 2023-10-09T00:00:00
---


# 1.2 스프링 애플리케이션 초기 설정하기(부트스트랩)

@SpringBootApplication 애노테이션의 역할.

- @SpringBootConfiguration : 현재 클래스(XXXApplication)을 config 클래스로 지정.(=@Configuration)
- @EnableAutoConfiguration : 스프링 부트 자동 구성 활성화.
- @ComponentScan : 컴포넌트 검색 활성화. 자동으로 @Component, @Controller, @Service 등의 클래스를 스프링 애플리케이션 컨텍스트에 컴포넌트로 등록.

<br>

<br>

# 2.4 간단한 컨트롤러(뷰 컨트롤러) 정의

뷰에 요청을 전달하기만 하는 컨트롤러를 선언하는 방법.

```java
package clowoodive.example.book.springinaction.taco.web;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

어떤 구성 클래스에서도 `WebMvcConfigurer` 인터페이스를 구현하고 오버라이딩 할 수 있음.

```java
package clowoodive.example.book.springinaction;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@SpringBootApplication
public class SpringinactionApplication implements WebMvcConfigurer {

    public static void main(String[] args) {
        SpringApplication.run(SpringinactionApplication.class, args);
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

<br>

<br>

# 2.5 뷰 템플릿 라이브러리

Thymeleaf등 자유롭게 선택 할 수 있으며 JSP(JavaServer Pages)의 경우는 톰캣이나 제티 서블릿 컨테이너 자체에서 제공하므로 의존성을 추가할 필요 없음.

만약 JSP를 선택한다면, /WEB-INF 밑에서 JSP 코드를 찾기에 JAR가 아닌 WAR로 패키징 해야 함.

<br>

<br>

# 2.5.1 템플릿 캐싱

애플리케이션을 다시 시작하지 않고 변경된 템플릿을 새로고침으로 확인하려면, 디폴트 true로 구성된 템플릿 캐싱 속성을 false로 변경해야 함. 단 실서비스에서는 원래대로 돌릴 것.

애플리케이션 실행 중에 html 파일을 변경하고 Recompile하면 새로고침으로 확인 할 수 있음.

```groovy
spring.thymeleaf.cache=false
```