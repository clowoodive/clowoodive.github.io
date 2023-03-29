---
title: "[Spring] Multiple/Duplicate form submission 방지"
excerpt: ""

categories:
  - Spring
tags:
  - Spring Security
  - Thymeleaf
  - CSRF
  - MVC
  - Multiple Submission
  - Interceptor
  - Filter
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


백오피스 웹서버에서 한 유저의 form POST 요청이 중복해서 발생하는 경우가 생겼다. 
로그를 조회 해 보니 수백 ms 내에 두번의 요청이 들어왔고, 발생 경로가 특이한 것도 아니어서 전송 지연 또는 double click 정도를 의심 해 볼 수 있었다.
소수의 유저가 사용하고 발생 빈도도 서비스 기간에 비해 낮은 편이지만 그대로 둘 수는 없어서 방법을 찾아보니 아래와 같은 것들이 있었다.

- javascript로 제출 버튼 비활성화
- Post-Redirect-Get 패턴 사용
- client-sever 간 unique token 사용

해당 서비스는 javascript를 사용하지 않고 PRG 패턴이 적용 되었음에도 문제가 발생했기에 unique token 방식으로 방향을 잡았다.

Spring Security의 CSRF 처리 방식과 유사하게 unique token을 사용하는 filter 처리 방법과 간단하게 interceptor에서 요청 시간을 제한하는 방법 두가지를 시도해 본 것을 정리한다. 

미리 알아둬야 할 것은 이 두 방법 모두 동일한 요청이 동시에 들어오는 경우 제대로 처리되는지는 보장하지 못한다. 이유는 10ms 이내의 10개 스레드 호출 테스트를 30회 정도 반복하면 첫 테스트를 포함해서 확률적으로 간혹 실패하기 때문인데 테스트 프로세스 초기에 부하가 발생해서 그런 것으로 추측된다.

<br><br>

# Unique Token 방식

Spring Security의 CSRF 방지 토큰 적용에 대해 지난 포스팅에서 알아봤었는데, 이와 유사한 방식으로 custom filter를 등록해서 Form Multiple Submit(FMS) 라 명명한 토큰을 form에 삽입하고 POST 요청시 토큰 검증을 통해 요청을 처리/거절 하는 방식이다.
Spring Security의 CSRF와 다른 점은 form의 GET 요청시 항상 새로운 토큰이 발급되고 해당 form 제출시에 토큰은 만료된다는 것이다.

## FmsFilter 구현

CsrfFilter의 코드를 베이스로 구현했는데, CsrfFilter에서는 POST 요청이 오더라도 매번 토큰을 갱신하지 않기에 `doFilterInternal` 메소드를 수정해서 POST/PUT/DELETE 요청인 경우 토큰을 재발급 하게 수정했다.

```java
@Component
public class FmsFilter extends OncePerRequestFilter {
		
    ...
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        request.setAttribute(HttpServletResponse.class.getName(), response);

        ...

        String fmsToken = null;
        HttpSession session = request.getSession(false);
        if (session != null) {
            fmsToken = (String) session.getAttribute(FMS_PARAMETER_NAME);
            if (fmsToken == null) {
                fmsToken = UUID.randomUUID().toString();
                session.setAttribute(FMS_PARAMETER_NAME, fmsToken);
            }
        }
        request.setAttribute(FMS_PARAMETER_NAME, fmsToken);

        if (!this.requireFmsProtectionMatcher.matches(request)) {
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Did not protect against FMS since request did not match "
                        + this.requireFmsProtectionMatcher);
            }
            filterChain.doFilter(request, response);
            return;
        }

        var actualToken = request.getParameter(FMS_PARAMETER_NAME);
        if (equalsConstantTime(fmsToken, actualToken)) {
            if (session != null) {
                var newFmsToken = UUID.randomUUID().toString();
                session.setAttribute(FMS_PARAMETER_NAME, newFmsToken);
                request.setAttribute(FMS_PARAMETER_NAME, newFmsToken);
            }
        } else {
            this.logger.debug(
                    LogMessage.of(() -> "Invalid FMS token found for " + UrlUtils.buildFullRequestUrl(request)));
            response.sendError(HttpStatus.NOT_ACCEPTABLE.value(), HttpStatus.NOT_ACCEPTABLE.getReasonPhrase());
            return;
        }
        filterChain.doFilter(request, response);
    }

    ...

}
```

코드는 대략 아래와 같은 흐름으로 구현했다.
- 토큰이 없으면 토큰을 생성하고 session과 request에 저장
- POST/PUT/DELETE 요청이 아니면 다음 filterChain으로 넘어감
- session과 request의 토큰이 같으면 새로운 토큰을 생성해서 갱신하고, 다를 경우 NOT_ACCEPTABLE(406) 을 반환함

## RequestDataValueProcessor bean 재정의

request에 저장된 CSRF 토큰은 Thymeleaf에서 form을 생성하면서 `CsrfRequestDataValueProcessor` 클래스의 `getExtraHiddenFields` 메소드를 호출해서  hidden 필드로 form에 삽입한다. 
custom FMS 토큰을 동일하게 처리 할 방법을 찾지 못해서 이 `CsrfRequestDataValueProcessor` 빈을 재정의 해서 등록하는 방식으로 처리했다.

```java
@Component
public class CsrfWithFmsRequestDataValueProcessor implements RequestDataValueProcessor {

    ...

    @Override
    public Map<String, String> getExtraHiddenFields(HttpServletRequest request) {
        if (Boolean.TRUE.equals(request.getAttribute(this.DISABLE_CSRF_TOKEN_ATTR))) {
            request.removeAttribute(this.DISABLE_CSRF_TOKEN_ATTR);
            return Collections.emptyMap();
        }
        CsrfToken token = (CsrfToken) request.getAttribute(CsrfToken.class.getName());
        if (token == null) {
            return Collections.emptyMap();
        }
        Map<String, String> hiddenFields = new HashMap<>(1);
        hiddenFields.put(token.getParameterName(), token.getToken());

        // request의 FMS 토큰을 hidden field에 포함
        var fmsToken = (String) request.getAttribute(FmsFilter.FMS_PARAMETER_NAME);
        if(fmsToken != null)
            hiddenFields.put(FmsFilter.FMS_PARAMETER_NAME, fmsToken);

        return hiddenFields;
    }

    @Override
    public String processUrl(HttpServletRequest request, String url) {
        return url;
    }

    ...

}
```

request에서 FMS 토큰을 추출해서 `hiddenFields` map에 담는 부분(주석 아래 3줄)만 추가하고 나머지 코드는 변경하지 않았다.

그리고 기존 `CsrfRequestDataValueProcessor` 대신 수정한 processor가 주입되도록 아래와 같이 적당한 config 파일에 bean을 재정의 했다.

```java
@Bean
RequestDataValueProcessor requestDataValueProcessor() {
    return new CsrfWithFmsRequestDataValueProcessor();
}
```

테스트 코드로 확인 해 보면 같은 FMS 토큰으로 POST 요청 시 406 코드가 반환되고, 5ms 이상의 간격으로 들어오는 POST 요청을 FMS 토큰으로 잘 필터링 해준다.

그리고 `BeanDefinitionOverrideException` 을 피하기 위해서는 Spring 5.1이 반영된 Spring Boot 2.1 버전 부터 아래와 같이 세팅이 필요하다.

```bash
spring.main.allow-bean-definition-overriding=true
```

<br><br>

# Interceptor에서 동일 요청 시간 제한 방식

동일한 method와 URI로 들어오는 POST 요청들을 단순히 이전 요청에서 특정 시간 이후에만 받아주는 방식이다.

## Interceptor 구현

```java
@Component
public class Interceptor implements HandlerInterceptor {

    private final Log logger = LogFactory.getLog(getClass());
    private final int denyGapTimeMs = 300;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.debug("preHandle, " + request.getRequestURI() + ", " + request.getQueryString());

        HttpSession session = request.getSession(false);
        if (session != null && request.getMethod().equals("POST")) {
            String oldMethod = (String) session.getAttribute("req_method");
            String oldPath = (String) session.getAttribute("req_path");
            Long oldTime = (Long) session.getAttribute("req_time");

            long curMillis = System.currentTimeMillis();

            if (Strings.isEmpty(oldMethod) || Strings.isEmpty(oldPath) || oldTime == null
                    || !request.getMethod().equals(oldMethod) || !request.getRequestURI().equals(oldPath)) {
                session.setAttribute("req_method", request.getMethod());
                session.setAttribute("req_path", request.getRequestURI());
            } else {
                long gap = curMillis - oldTime;
                if (gap < denyGapTimeMs) {
                    logger.warn(oldTime + " - " + curMillis + " = " + gap + " millis");
                    throw new ResponseStatusException(HttpStatus.TOO_MANY_REQUESTS, "same request was received in a short time");
                }
            }
            session.setAttribute("req_time", curMillis);
        }

        return true;
    }
}
```

session에 method, uri, 요청시간을 저장해서 다음 요청 시 method와 uri가 동일하다면 시간을 비교하고 특정 시간(300ms) 이내이면 TOO_MANY_REQUESTS(429)를 리턴한다.

## Interceptor 등록

interceptor 설정은 아래와 같이 간단하게 구성했다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private Interceptor interceptor;

    @Autowired
    public WebConfig(Interceptor interceptor) {
        this.interceptor = interceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(interceptor).addPathPatterns("/**");
    }
}
```

테스트를 통해 확인하면 지정한 300ms 이내의 동일한 POST 요청은 모두 거절하는 것을 확인 할 수 있다.
시간을 제한하는 방식을 인터셉터에서 구현했지만 필터에서도 세션 처리가 가능하기에 더 앞단인 필터에서 구현해도 된다.

앞서 말한 것처럼 같은 시점에 동일한 요청은 멀티 스레딩으로 인해 세션을 통해 검사하는 코드들이 동시에 실행 될 수 있으므로 보장하기 어렵고, 이는 테스트 코드를 통해서도 확인 할 수 있다.
따라서 의도적인 동시 호출이 아니라 수십ms 이상의 시간차로 요청이 들어오는 더블 클릭과 같은 상황에서 앞단에 적용 해 볼 만 하다.

코드는 [여기](https://github.com/clowoodive/pilot/tree/main/pilot-springboot-mvc-multiple-submit-filter).

CSRF를 Thymeleaf에 적용하는 포스팅은 [여기](https://clowoodive.github.io/spring/post-spring-security-csrf-thymeleaf).

### References

- [https://terasolunaorg.github.io/guideline/5.0.0.RELEASE/en/ArchitectureInDetail/DoubleSubmitProtection.html](https://terasolunaorg.github.io/guideline/5.0.0.RELEASE/en/ArchitectureInDetail/DoubleSubmitProtection.html)
- [http://junjun-java.blogspot.com/2014/12/spring-mvc-preventing-duplicate-form.html](http://junjun-java.blogspot.com/2014/12/spring-mvc-preventing-duplicate-form.html)
- [https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=agapeuni&logNo=60208814780](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=agapeuni&logNo=60208814780)