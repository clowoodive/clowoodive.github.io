---
title: "[Spring] Spring Security의 CSRF 토큰 Thymeleaf에 적용하기"
excerpt: ""

categories:
  - Spring
tags:
  - Spring Security
  - Thymeleaf
  - CSRF
last_modified_at: 2022-11-16T00:00:00
---

{% capture notice-env %}
#### Environment
	- Java 11
	- Spring Boot 2.5.12
	- Spring Security 5.5.5
    - Thymeleaf 3.0.15 RELEASE
	- Gradle 7.5.1 
{% endcapture %}
<div class="notice--primary">{{ notice-env | markdownify }}</div>


코드는 [여기](https://github.com/clowoodive/pilot/tree/main/pilot-spring-security-thymeleaf).


사용중인 Spring Security 5.5.5 버전은 적용하면 CSRF 방지 기능이 기본으로 활성화 되어 form 요청 시 자동으로 CSRF 토큰을 삽입 해 준다.

심플하게 프로젝트를 생성해서 적용하고 확인 해 보자.

여기선 다루지 않겠지만 Spring Security 5.7.0-M2 버전 부터는 세팅시 *`SecurityFilterChain`* 빈을 등록 하는 등 변경사항이 있으니  [이곳](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter) 을 참고.

먼저 필요한 의존성을 세팅 해주자.

build.gradle

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-security'
}
```

그리고 디버깅하기 쉽게 로그도 조정해주고,

application.properties

```bash
logging.level.org.springframework.web=TRACE  # WEB 하위 심각도 로깅 on
spring.mvc.log-request-details=true          # DEBUG/TRACE 수준의 민감한 세부 정보 표시
```

매번 구동 시 마다 임시 패스워드를 사용하기 귀찮으니 유저 세팅을 하고, `configure` 메소드 오버라이드를 통해 인증 관련 기본 세팅을 한다.

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withDefaultPasswordEncoder().username("user").password("1234").roles("USER").build());
        return manager;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests(authorize -> authorize
                        .anyRequest().authenticated()
                )
                .formLogin(withDefaults())
                .httpBasic(withDefaults());
    }
}
```

타임리프 form을 아래와 같이 간단히 만드는데, 이때 주의해야 할 것은 form의 action 태그를 꼭 타임리프의 속성 `th:action` 으로 해주어야 CSRF 토큰이 삽입된다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">

<body>

<div class="container">

    <form th:action="@{/filter/message}" method="post">
        <div class="form-group">
            <label for="message">인사</label>
            <input type="text" id="message" name="message" placeholder="인사말을 입력하세요"/>
        </div>
        <button type="submit">등록</button>
    </form>

</div>

</body>
</html>
```

form을 요청하는 controller를 생성 하고

```java
@GetMapping("/filter/message")
    public String createMessageForm() {
        return "message-form";
    }
```

웹 브라우저에서 호출 해 보면 개발자 도구로 아래와 같이 _csrf 토큰이 삽입된 것을 알 수 있다.

![security_thymeleaf1]({{ '/assets/images/security_thymeleaf1.png' | relative_url }}){: .align-center}

이 토큰의 value값이 다르면 post 요청을 거부하게 되는데, 간단히 확인해 보려면 input 필드 자체를 지워서 토큰을 없애고 post 요청하면 토큰이 검증되지않아 403 응답을 받게 된다.

![security_thymeleaf2]({{ '/assets/images/security_thymeleaf2.png' | relative_url }}){: .align-center}

Reference

[https://docs.spring.io/spring-security/site/docs/5.5.5/reference/html5/#jc](https://docs.spring.io/spring-security/site/docs/5.5.5/reference/html5/#jc)