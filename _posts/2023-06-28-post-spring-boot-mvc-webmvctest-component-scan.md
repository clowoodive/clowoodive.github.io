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
last_modified_at: 2023-06-28T00:00:00
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

`@WebMvcTest` 애노테이션으로 Controller를 테스트 하던 중 경로를 분리해서 적용중이던 custom filter 2개를 모두 통과하는 문제 발생.

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

## 1.1. Custom Filter 1

```java
@Component
public class AppOncePerRequestFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        ContentCachingRequestWrapper wrappingRequest = new ContentCachingRequestWrapper(request);

        chain.doFilter(wrappingRequest, response);
    }
}
```

## 1.2. Custom Filter 2

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

## 1.3. Custom Filter 등록

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

`@WebMvcTest` 에서 자동 구성 및 자동 구성 제외되는 항목에 따라 Filter를 구현한 Custom Filter 1과 2는 필터로 등록이 되었지만, `@Component` 애노테이션이 붙은 `FilterConfig` 는 등록되지 않았기 때문에 경로 설정이 되지 않아 모든 필터 통과.

`@SpringBootTest` 와 `@AutoConfigureMockMvc` 를 이용해서 모든 구성을 완전하게 로드 해서 테스트 할 수 있지만 단위테스트를 추구하기 위해 이 방법은 배제함.

## 2.1. @WebMvcTest 자동 구성

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

## 2.2. 자동구성 제외 애노테이션

- `@Component`
- `@ConfigurationProperties`
- `@Service`
- `@Repository`

자세한 자동구성 애노테이션은 [여기](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix.test-auto-configuration) 참조.

<br>

<br>

# 3. 자동 구성 되지 않는 Bean 컨텍스트에 추가

`includeFilters` 속성을 통해 필터를 등록하는 `FilterConfig` 클래스를 애플리케이션 컨텍스트에 추가 할 수 있으며, 다른 속성들로 자동 구성에서 제외하거나 자동구성 자체를 disable 시킬 수 있음.

```java
@WebMvcTest(value = AppController.class, includeFilters = {
//        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {WebEndpointProperties.class}),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {FilterConfig.class})
})
```

<br>

<br>

# 4. 자동 구성된 클래스가 의존하는 Bean 참조 문제

여기서 추가로 `FilterConfig` 에서 `actuatorFilterRegistrationBean` 을 구성하면서 의존하는 `WebEndpointProperties` 를 참조하지 못해서 빈 구성에 실패하는 문제가 발생.

위의 코드에서 주석처리 된 것처럼 하거나 아래와 같이 처리해도 해결되지 않음.

```java
@WebMvcTest(value = AppController.class, includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {WebEndpointProperties.class, FilterConfig.class})
})
```

## 4.1. Bean 등록을 통해 의존 Bean 참조 문제 해결

`@WebMvcTest` 와 함께 `@Import` 를 사용해서 해결하거나

```java
@Import(value = WebEndpointProperties.class)
```

Test 클래스 내에서 Bean 으로 등록하는 `@MockBean`/`@SpyBean` 을 용도에 따라 사용해서 해결 가능.

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

Spring Boot에서 제공하는 자동구성이 적용되는 `@WebMvcTest` 와 같은 테스트 애노테이션을 사용 할 때는 테스트가 정상 작동하더라도 이번의 경우처럼 구성이 잘못 되었을 수 있으므로, 자동구성 항목/ 자동구성 제외 항목을 잘 살펴보고 꼼꼼하게 세팅할 필요가 있음.

<!--

예제 코드가 적용된 프로젝트는 [여기](https://github.com/clowoodive/toy/tree/main/investing).

-->

### References

- [https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html)
- [https://taetaetae.github.io/2020/04/06/spring-boot-filter/](https://taetaetae.github.io/2020/04/06/spring-boot-filter/)