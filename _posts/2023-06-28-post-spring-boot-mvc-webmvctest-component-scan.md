---
title: "[Spring Boot - MVC] @WebMvcTest로 자동 설정 되지 않는 컴포넌트 스캔하기"
excerpt: ""

categories:
  - Spring
tags:
  - Spring Boot
  - MVC
  - WebMvcTest
  - ComponentScan
  - Filter
  - Test
last_modified_at: 2023-12-13T00:00:00
---

{% capture notice-env %}
#### Environment
 - Java 11
 - Spring 5.3.18
 - Spring Boot 2.5.12
 - Gradle 6.9.2 
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


# 1. 문제 발생

`@WebMvcTest` 애너테이션을 사용해서  Controller를 테스트 하던 중, 경로를 분리해서 적용 중이던 커스텀 필터 두개가 모두 적용되는 문제가 발생했다.

### 테스트 코드

```java

@WebMvcTest(value = AppController.class)
class MyControllerUnitTest {

    @Autowired
    MockMvc mockMvc;
	
    @Test
    void testPayment() {
        ...
        var resultActions = mockMvc.perform(
                post("/app/pay")
        );
        ...
    }
}
```

### 커스텀 필터 1 코드

필터 처리 시 MyService 의존성이 필요하다는 점이 중요하다.

```java
@Component
public class AppOncePerRequestFilter extends OncePerRequestFilter {

    @Autowired
    private MyService myService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        ContentCachingRequestWrapper wrappingRequest = new ContentCachingRequestWrapper(request);
        // using myService

        chain.doFilter(wrappingRequest, response);
    }
}
```

### 커스텀 필터 2 코드

```java
@Component
public class ActuatorOncePerRequestFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        // logging

        chain.doFilter(wrappingRequest, response);
    }
}
```

### 커스텀 필터 등록 코드

```java
@Component
public class FilterConfig {
    @Bean
    public FilterRegistrationBean<AppOncePerRequestFilter> appFilterRegistrationBean(AppOncePerRequestFilter appOncePerRequestFilter) {
        FilterRegistrationBean<AppOncePerRequestFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(appOncePerRequestFilter);
        registrationBean.addUrlPatterns("/app/*");
        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean<ActuatorOncePerRequestFilter> actuatorFilterRegistrationBean(
            ActuatorOncePerRequestFilter actuatorOncePerRequestFilter, WebEndpointProperties actuatorWebEndpointProperties) {
        FilterRegistrationBean<ActuatorOncePerRequestFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(actuatorOncePerRequestFilter);
        registrationBean.addUrlPatterns(actuatorWebEndpointProperties.getBasePath() + "/*");
        return registrationBean;
    }
}
```

<br>

<br>

# 2. 원인

우선 스프링부트에서 필터를 등록하는 방법은 필터를 구현한 클래스에 `@Component`를 붙이는 방법(URL 매핑 불가)과 `@ServletComponentScan + @WebFilter`(필터 순서 지정 불가) 방식 그리고 `FilterRegistrationBean` 를 사용하는 방식이 있다. 

본문의 코드는 `FilterRegistrationBean` 을 사용함에도 필터 클래스에서 의존성 주입이 필요해서  `@Component` 가 붙어있는데 이 부분을 주의해야 한다.(필터는 한번만 등록되긴 한다.)

실제 실행이나 `@SpringBootTest` 를 사용하는 통합 테스트와 다르게 `@WebMvcTest` 만 사용하는 환경에서는 `@Component` 에 의해 필터는 자동으로 등록되지만 `@Component`가 달린 FilterConfig 내부의 `FilterRegistrationBean` 빈은 자동으로 구성하지 않는다. 따라서 URL 매핑이 구성되지 않기에 필터가 모든 요청에 적용되는 것이다.

`@WebMvcTest` 에 의해 자동 구성되는 항목들은 아래를 참고하자.

## 자동 구성

- `@Controller`
- `@ControllerAdvice`
- `@JsonComponent`
- `Converter`
- `GenericConverter`
- `Filter`
- `HandlerInterceptor`
- `WebMvcConfigurer`
- `WebMvcRegistrations`
- `HandlerMethodArgumentResolver`

## 자동 구성 제외

- `@Component`
- `@ConfigurationProperties`
- `@Service`
- `@Repository`

자세한 자동 구성 애너테이션은 [여기](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix.test-auto-configuration) 참조.

<br>

<br>

# 3. 수동 필터 등록으로 해결

`@WebMvcTest` 대신 `@SpringBootTest` 와 `@AutoConfigureMockMvc` 를 이용해 모든 구성을 완전하게 로드 해서 테스트 할 수 있지만 단위테스트를 추구하기 위해 이 방법은 배제한다.

그대신 아래 코드처럼 `@WebMvcTest` 의 `includeFilters` 속성을 통해 `FilterConfig` 클래스를 애플리케이션 컨텍스트에 추가 할 수 있으며, 이외에도 다른 속성들을 사용해서 자동 구성에서 제외하거나 자동구성 자체를 disable 시킬 수 있다.

```java
@WebMvcTest(value = AppController.class, 
includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {FilterConfig.class})
})
class MyControllerUnitTest {
    ...
}
```

<br>

<br>

# 4. 자동 구성된 클래스가 의존하는 Bean 참조 문제

만약 이 글의 예제 코드처럼 `FilterConfig` 에서 `actuatorFilterRegistrationBean` 을 구성하면서 의존하는 `WebEndpointProperties` 같은 객체가 있을 경우 빈 구성에 실패하는 문제가 발생 할 수 있다.

이때에는 아래처럼 `@Import` 를 사용해서 해결하거나

```java
@WebMvcTest(value = AppController.class, 
includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {FilterConfig.class})
})
@Import(value = {WebEndpointProperties.class}) // 또는 @MockBean 사용
class MyControllerUnitTest {
    ...
}
```

Test 클래스 내에서 Bean 으로 등록하는 `@MockBean`/`@SpyBean` 을 사용해서 의존성을 주입 해줄 수 있다.

```java
@SpyBean // 또는 @MockBean
WebEndpointProperties webEndpointProperties;

@Test
void testPayment() {
    ...
    var resultActions = mockMvc.perform(
            post("/app/pay")
    );
    ...
}
```

<br>

<br>

# 5. 정리

Spring Boot에서 제공하는 자동구성이 적용되는 `@WebMvcTest` 와 같은 테스트 애너테이션을 사용 할 때는 테스트가 정상 작동하더라도 이번의 경우처럼 구성이 잘못 되었을 수 있으므로, 자동구성 항목/ 자동구성 제외 항목을 잘 살펴보고 꼼꼼하게 세팅할 필요가 있다.

한번더 당부하자면 의존성 문제가 없는 경우 `FilterRegistrationBean` 으로 필터를 등록 할 때에는 필터 클래스에 `@Component` 가 필요하지 않다.

<!--

예제 코드가 적용된 프로젝트는 [여기](https://github.com/clowoodive/toy/tree/main/investing).

-->

### References

- [https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html)
- [https://taetaetae.github.io/2020/04/06/spring-boot-filter/](https://taetaetae.github.io/2020/04/06/spring-boot-filter/)
- [https://jronin.tistory.com/124](https://jronin.tistory.com/124)
- [https://www.baeldung.com/spring-boot-add-filter](https://www.baeldung.com/spring-boot-add-filter)
- [https://iseunghan.tistory.com/418](https://iseunghan.tistory.com/418)